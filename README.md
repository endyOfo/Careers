 Careers Module 
<a id="top"></a>
[![N|magento](https://cldup.com/dTxpPi9lDf.thumb.png)](https://magento.com/)</br>
Author: Endy Ofo </br>
</br>

Custom Magneto module to allow advertising of job vacancies. Content management users can add/delete or edit job vacancies to the business website. Potential applicants can view and apply for vacancies. 

# Table of Contents
  * [Prerequisites](#prerequisites)
  * [Test data installation script](#test-data-installation-script)
  * [DATA MODEL DESIGN](#data-model-design)
    * [Entity Tables](#entity-tables)
        * [1. ubt_careers_vacancy](#ubt_careers_vacancy)
        * [2. ubt_careers_officelocation](#ubt_careers_officelocation)
        * [3.ubt_careers_vacancystatus](#ubt_careers_vacancystatus)
        * [4.ubt_careers_vacancytype](#ubt_careers_vacancytype)
        * [5.ubt_careers_viewablelist](#ubt_careers_viewablelist)
        * [6.ubt_careers_configvalues](#ubt_careers_configvalues)
        * [7. ubt_careers_storeview](#ubt_careers_storeview)
        * [8. ubt_careers_thirdpartysites](#ubt_careers_thirdpartysites)
        * [9. ubt_careers_permittedfileextensions](#ubt_careers_permittedfileextensions)
        * [10. ubt_careers_jobapplicants](#ubt_careers_jobapplicants)
  * [BACKEND DESIGN](#backend-design)
    * [Manage Vacancies](#manage-vacancies)
    * [Career Settings (system config)](#career-settings)
  * [FRONTEND DESIGN](#frontend-design)
    * [Main page](#main-page)
    * [Detailed vacancy page](#detailed-vacancy-page)
 * [FUTURE TASK/ IMPROVEMENTS](#future-tasks)
<a id="prerequisites"></a>

# Prerequisites  

1. Installed magento community 1.9.2 + /Enterprise 1.14 +
1. PHP 5.5 +
1. mySQl 5.6 + 


# Installation:-
Manual installation. 
Download and unzip the package, place the file and folders into the corresponding folders

  - app\code\local\Ubt\Careers
  - app\etc\modules\Ubt_Careers.xml 
  - app\design\adminhtml\default\default\layout\ubt_careers.xml
  - app\design\frontend\base\default\layout
  - app\design\frontend\base\default\template\ubt\careers
  - media\careers
  - skin\frontend\base\default\css
  - skin\frontend\base\default\font
  - skin\frontend\base\default\js

### **Activate Module-** 
Once the above step is completed, activate the module by clearing the caches at the location below. 

**NOTE:**  Clearing magneto’s  cache will trigger all newly installed modules to run their installation scripts thereby installing all required data for that module.  

<a id="cache-clear"></a>
### **Cache Clear** 
Login into the admin panel and follow the links below. 

```sh
admin => system => Cache management
```
Select all - then click the  'Flush Magneto Cache' button. 

<a id="test-data-installation-script"></a>
# Test Data Installation Script
**FOR DEVELOPMENT MODE ONLY -**

The 'data installation script' will install test data if activated.

The script is located here: 

```sh
app\code\local\Ubt\Careers\data\ubt_careers_setup\data-install-0.1.0.php
```

The script is commented out by default.

In order to activate it the following procedure should be followed.

- Go to the script page listed above and uncomment it by removing the following characters at the top and bottom of the page respectively:    /**      AND       **/
-  Activating the test script now depends on whether the 'careers module' has already been activated. If not, then 'the installation script' will automatically install the test data once the actual module is activated. 
-  If however the module has already been activated, then the simplest way to install the data is to force a   **pseudo re-activation** of the module.

 **Pseudo Re-activation Of A Module** </br>
 
This procedure does not affect the module or its existing data in any way, all it does is force magneto to run through the module's 'installation scripts' again.  

Go to the "MySQL table" or whichever data storage table is being used, open the table: **'core_resource'**
Then search through the table, find and delete the entire row for the following entry: **ubt_careers_setup**

Once the row is deleted run the [Cache Clear](#cache-clear) again and the script will install the data.  


<a id="data-model-design"></a>

[back to top](#top)</br>

# DATA MODEL DESIGN:-

The script for the module's data entities creation is stored here:   

```sh
app\code\local\Ubt\ManageStaff\sql\ubt_managestaff_setup\mysql4-install-0.1.0.php
```

The module consists of 10 data entities:

1.  **[ubt_careers_vacancy](#ubt_careers_vacancy)**  
1. 	**[ubt_careers_officelocation](#ubt_careers_officelocation)**
1. 	**[ubt_careers_vacancystatus](#ubt_careers_vacancystatus)**
1. 	**[ubt_careers_vacancytype](#ubt_careers_vacancytype)**
1. 	**[ubt_careers_viewablelist](#ubt_careers_viewablelist)**
1.  **[ubt_careers_configvalues](#ubt_careers_configvalues)**
1. 	**[ubt_careers_storeview](#ubt_careers_storeview)**
1. 	**[ubt_careers_thirdpartysites](#ubt_careers_thirdpartysites)**
1. 	**[ubt_careers_permittedfileextensions](#ubt_careers_permittedfileextensions)**
1.  **[ubt_careers_jobapplicants](#ubt_careers_jobapplicants)**

The entity models are all located within the following folder: 

```sh
app\code\local\Ubt\Careers\Model
```

    
 **1 - ubt_careers_vacancy** </br>
This is the main data entity [table](#ubt_careers_vacancy) for the module and contains all data relevant to a particular vacancy. It is reliant on some of the other entities to the extent that it requires look up for some of its columns. For example, the column **'job_location'** only contains the Id of partiular locations. However, in order to get the actual location name  we need to look up its value in the corresponding column (**location_id**) of the  **ubt_careers_officelocation**.

An example of how the tables are linked during rendering can be seen in the vacancy grids $this->_prepareCollection() method: 

```php
app\code\local\Ubt\Careers\Block\Adminhtml\Vacancy\Grid.php
protected function _prepareCollection()
{
    $collection  = Mage::getModel('ubt_careers/vacancy')->_getVacancyCollection();
}
```

```php
app\code\local\Ubt\Careers\Model\vacancy.php

public function _getVacancyCollection($id=null)
{
    $collection = $this->getCollection(); 
    $collection  = Mage::getModel('ubt_managestaff/vacancy')->getCollection(); 
    $vacancytype = Mage::getSingleton('core/resource')->getTableName('ubt_careers/vacancytype'); 
    $vacancystatus  = Mage::getSingleton('core/resource')->getTableName('ubt_careers/vacancystatus'); 
    $location       = Mage::getSingleton('core/resource')->getTableName('ubt_careers/officelocation'); 
    $applications   = Mage::getSingleton('core/resource')->getTableName('ubt_careers/jobapplicants'); 
        
    $collection->getSelect()->group('main_table.vacancy_id')
                   ->joinLeft(array('vacancytype'=>$vacancytype),'`main_table`.`vacancy_type` = `vacancytype`.`vacancytype_id`',array('vacancytype'))
                   ->joinLeft(array('vacancystatus'=>$vacancystatus),'`main_table`.`vacancy_status` = `vacancystatus`.`vacancystatus_id`',array('vacancystatus'))
                   ->joinLeft(array('location'=>$location),'`main_table`.`job_location` = `location`.`location_id`',array('officelocation'));
         
    return $collection;   
}
```

**2 - ubt_careers_officelocation** </br>
This [table](#ubt_careers_officelocation) contains the names/postcodes of UBT  office locations e.g London cv34. It acts as a look up table for the **job_location** column in the ubt_careers_vacancy table. 

**3 - ubt_careers_vacancystatus**</br>
This [table](#ubt_careers_vacancystatus) contains the names/values of the different type of status  vacancy may have e.g 'filled'. The table acts as a look up for the **vacancy_status** column in the ubt_careers_vacancy table. 

**4 - ubt_careers_vacancytype**</br>
This [table](#ubt_careers_vacancytype) contains the names of the different type of vacancies egg 'full-time'. The table acts as a look up for the **vacancy_type** column in the ubt_careers_vacancy table. 

**6 - ubt_careers_viewablelist**</br>
This [table](#ubt_careers_viewablelist) contains the different types of viewable settings for staff members’ e.g. 'access to all logged in members'. The table acts as a look up for the **content_viewable** column in the ubt_careers_vacancy table. 

The attributes of these models and the relationships between them are shown below.

<a id="entity-tables"></a>

[back to top](#top)</br>

# Entity Tables 
</br>


<a id="ubt_careers_vacancy"></a>
### **1. Table: ubt_careers_vacancy**
:- Main Table For The Ubt Careers Module. 

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
| vacancy_id    | int(10) |     | 5    | Primary key,auto increment |    |
| job_title    | text   | title of job   |varchar(250)   | Regional manager    | required    |
| url_key       | text   |  surname | varchar(250)  |  doe    | required   |
|lowersalary_range |smallint(5)	    |  salary range-starting amount  | 5 (int)  |  20    |  |
|higher salary_range| smallint(5)	   |salary range-maximium amount  | 5 (int)  |  50| |
|vacancy_type| smallint(5)  | the Id for vacancytype_id column of ubt_careers_vacancytype table | 5(int)  | 6   || 
|job_location   |smallint(5) |  the Id of location_id column of ubt_careers_officelocation table  | 5(int)   |  5    | 
|application_deadline |datetime | deadline for applying for vacancy  | datetime | 2017-12-12   |     |
|vacancy_body | text    | vacancy descriptions  | varchar(850)  | the job is located in london    |     |
|status         |smallint(5)	    |  234  | 5   |  1    |     |
|teamcategory_id|smallint(5)	    | service code the staff member affiliated with-linked to service table  | 5   |  1    |     |
|content_viewable|smallint(5)	   |   the Id for the viewablelist_id column of ubt_careers_viewablelist table | 5   |  1    |     |
|vacancy_status|smallint(5)	   |  the Id for the vacancystatus_id column of ubt_careers_vacancystatus table                  | 5   |  1    |     |
| created_at     |timestamp |time row was created   | timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp  | time row was amended   | timestamp  | 2017-10-12 09:56:19 |  |

<a id="ubt_careers_officelocation"></a>

### **2. Table : ubt_careers_officelocation**

look up table for the  "job_location" column

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
| location_id   | int(10) |     | 5    | Primary key,auto increment |    |
| officelocation  | text   | country abbriviation   |varchar(250)   |  UK   | required    |
| full_name     | text   | full country name | varchar(250)  |  United Kingdom| required   |
| status       | smallint(5)    |    | 5   |  1    |  |
| store_id      | smallint(5)  |  | 5 |  1    |     |
| created_at     |timestamp |time row was created   | timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp  | time row was amended   | timestamp  | 2017-10-12 09:56:19 |  |

<a id="ubt_careers_vacancystatus"></a>


###  **3. Table: ubt_careers_vacancystatus**

Look up table for the "vacancy_status" column

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
|vacancystatus_id | int(10) |     | 5    | Primary key,auto increment |    |
|vacancystatus  | text   | full name of service | varchar(250)  | service_name| required   |
| created_at     |timestamp |time row was created   | timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp  | time row was amended   | timestamp  | 2017-10-12 09:56:19 |  |

<a id="ubt_careers_vacancytype"></a>



### **4. Table: ubt_careers_vacancytype**

Look up table for the "vacancy_type" column

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
|vacancytype_id  | int(10) |     | 5    | Primary key,auto increment |    |
| vacancytype| text   | full name of service | varchar(250)  | service_name| required   |
| status       | smallint(5)    |    | 5   |  1    |  |
| store_id      | smallint(5)  |  | 5 |  1    |     |
| created_at     |timestamp |time row was created   | timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp  | time row was amended   | timestamp  | 2017-10-12 09:56:19 |  |

<a id="ubt_careers_viewablelist"></a>


### **5. Table: ubt_careers_viewablelist**

Look up table for the "content_viewable" column

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
|viewablelist_id | int(10) |     | 5    | Primary key,auto increment |    |
|viewable_name| text   | full name of service | varchar(250)  | service_name| required   |
| status       | smallint(5)    |    | 5   |  1    |  |
| store_id      | smallint(5)  |  | 5 |  1    |     |
| created_at     |timestamp |time row was created   | timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp  | time row was amended   | timestamp  | 2017-10-12 09:56:19 |  |

<a id="ubt_careers_configvalues"></a>
[back to top](#top)</br>
### **6. Table: ubt_careers_configvalues**</br>

Look up table for the "vacancy_type" column

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
|vacancytype_id  | int(10) |     | 5    | Primary key,auto increment |    |
| vacancytype| text   | full name of service | varchar(250)  | service_name| required   |
| status       | smallint(5)    |    | 5   |  1    |  |
| store_id      | smallint(5)  |  | 5 |  1    |     |
| created_at     |timestamp |time row was created   | timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp  | time row was amended   | timestamp  | 2017-10-12 09:56:19 |  |

<a id="ubt_careers_thirdpartysites"></a>

### **8. Table: ubt_careers_thirdpartysites**</br>

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
|thirdparty_id | int(10) |     | 5    | Primary key,auto increment |    |
|vacancy_id    | smallint(5)  |  | 5 |  1    |     |
|site_name  | text   | full name of service | varchar(250)  | service_name| required   |
|url_jobpost  | text   | full name of service | varchar(250)  | service_name| required   |
| status       | smallint(5)    |    | 5   |  1    |  |
| created_at     |timestamp |time row was created   | timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp  | time row was amended   | timestamp  | 2017-10-12 09:56:19 |  |


<a id="ubt_careers_permittedfileextensions"></a>


### **9. Table: ubt_careers_permittedfileextensions**</br>

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
|type_id | int(10) |     | 5    | Primary key,auto increment |    |
|extension  | text   | full name of service | varchar(250)  | service_name| required   |
| created_at     |timestamp    |   | 12   | 2017-10-12 09:56:19|     |
| update_at     |timestamp   |    | 12   | 2017-10-12 09:56:19 |  |
| status       | smallint(5)    |    | 5   |  1    |  |
| status       | smallint(5)    |    | 5   |  1    |  |

[back to top](#top)</br>
<a id="ubt_careers_jobapplicants"></a>


### **10 . Table: ubt_careers_jobapplicants**</br>

 Records details of all applicants who applied for the vacancies. It's column 	'**vacancy_id**’ is dependent on looking up the  '**vacancy_id**'  column of  ubt_careers_vacancy to get the vacancy title.  
 

| Column Name     | column Type | Description   | Value Range   |Example Data    |Notes   |
| :------- | ----: | :---: |:---: |:---: |:---: |
|applicant_id| int(10) |     | 5    | Primary key,auto increment |    |
|vacancy_id  | text   | full name of service | varchar(250)  | service_name| required   |
|first_name  | text   | first name   |varchar(250)   |  john    | required    |
|surname     | text   |  surname | varchar(250)  |  doe    | required   |
|uploaded_cv | text    |  234  | varchar(250)   |  Senior Developer    | required |
|email       | text    | job title  | varchar(250)   |  5    | required    |
|phone_number| text    | staff profile  | varchar(250)   |  5    |     |
|message     | text   |  staff image  | varchar(250)   |  5    |     |
|store_id    |smallint(5)	    |  234  | 5   |  1    |     |
|status      |smallint(5)	    |  234  | 5   |  1    |     |
| created_at     |timestamp |time row was created   | timestamp  | 2017-10-12 09:56:19|     |
| update_at     |timestamp  | time row was amended   | timestamp  | 2017-10-12 09:56:19 |  |


[back to top](#top)</br>
<a id="backend-design"></a>

# BACKEND DESIGN:-   

The backend user interface is accessed via a “Careers” menu. This has been added to the existing CMS menu in the Magneto admin pages.

The 'Careers menu' has 2 options:  

**A. Manage Vacancies** </br>
**B. Career Settings (system config)**</br>

<a id="manage-vacancies"></a>

## A.  The 'Manage Vacancies' Option:-

This option leads to the page responsible for rendering the grid iteration of all vacancies. There is a button at the top of page that clicks onto the 'add new vacancy forms'.

Each iteration in the grid is clickable and leads to a form for editing that team vacancies data.

#### RELEVANT LINKS FOR 'Manage Vacancies' OPTIONS:- 
 
The **controller** for this option (i.e. the grid and the 'add new vacancy forms') is located here:

```php
app\code\local\Ubt\Careers\controllers\Adminhtml\Managevacancy\Vacancy\VacancyController.php
public function indexAction(){
}    
```

The **grid page** is located here: 
```sh
The grid container: 
app\code\local\Ubt\Careers\Block\Adminhtml\Vacancy.php
```
```sh
The grid page: 
app\code\local\Ubt\Careers\Block\Adminhtml\Vacancy\Grid.php
```

The **xml page** for the grid page is located at

```sh
app\design\adminhtml\default\default\layout\ubt_careers.xml
```
The code below is an example of how the xml page acts as a links between the **front end** and the **database logic**..
```html
   <adminhtml_managevacancy_vacancy_vacancy_index>
        <reference name="content">
            <block type="ubt_careers/adminhtml_vacancy" name="careermanager" />
        </reference>
    </adminhtml_managevacancy_vacancy_vacancy_index>
```

#### The 'add new vacancy' forms 

The container for the forms are located here: 

```sh
Form container: 
app\code\local\Ubt\Careers\Block\Adminhtml\Vacancy\Edit.php
```
```sh
Form Tabs buttons (the right sided tab buttons on the form pages): 
app\code\local\Ubt\Careers\Block\Adminhtml\Vacancy\Edit\Tabs.php
```

The actual forms pages are located here: 

>app\code\local\local\Ubt\Careers\Block\Adminhtml\Vacancy\Edit\Tab\Form.php
>app\code\local\local\Ubt\Careers\Block\Adminhtml\Vacancy\Edit\Tab\Content.php
>app\code\local\local\Ubt\Careers\Block\Adminhtml\Vacancy\Edit\Tab\Status.php


Once the save button is clicked the forms are submitted to the **save method** of the StaffController.


**Save Method And Form Validation;** 
```php
app\code\local\Ubt\Careers\controllers\Adminhtml\Managevacancy\Vacancy\VacancyController.php

public function saveAction() {
   if (($preFilteredData =  $this->_validateParams($this->getRequest()->getPost('team'))) &&    ($this->_validateFormKey())){ 
}

//validate form values prior to saving them. 
protected function _validateParams($params) {
}  
```

<a id="career-settings"></a>
[back to top](#top)</br>
## B.  Career Settings (system config):-

This option leads to the page responsible for rendering the tabbed grid iterations of each of the following entities:

1. Team Category Setting
1. Region Setting
1. Service Setting
1. Viewable Setting  

Each entity is hosted within its own tab and each iteration within that tab is clickable and leads to a form for editing the relevant data.

#### RELEVANT LINKS FOR 'Team Config' OPTIONS:- 

The **controller** for this option (i.e. the grids and the 'add new forms') is located here:

```sh
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Modulesettings\SettingsController.php
```

The  **grid pages** are located here: 
```sh
The Form Container: 
app\code\local\Ubt\ManageStaff\Block\Adminhtml\config\edit.php
```

```sh
The grid pages: 
app\code\local\Ubt\ManageStaff\Block\Adminhtml\config\edit\tabs.php
```

**NOTE:** The grids uses a form container rather than the traditional 'grid container', thus enabling access to the **'forms tabs'**. The advantage of the form container over the grid container is that it enables the rendering of several grids on the same page.

The index method of the SettingsController.php joins all the grids under a single tab: 


```php
 public function indexAction(){
    $block = $this->getLayout()
                ->createBlock('ubt_managestaff/adminhtml_config_edit')
                .......
                ->_addLeft($this->getLayout()->createBlock('ubt_managestaff/adminhtml_config_edit_tabs'))
}    
```
**Rendering Of The Individual Grids:-**

The individual grids are all stored in the following location:

```sh
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Config\Grids
```

[back to top](#top)</br>
#### Example of how the grids are rendered:

**FIRST- The Edit Tab**    (adds tab buttons to the left side of the grid page.):

```php
app\code\local\Ubt\ManageStaff\Block\Adminhtml\config\edit\tabs.php
//example of the TEAMCATEGORY tab.
$this->addTab('form_teamcategory', array(
   'label'     => Mage::helper('ubt_managestaff')->__('Team Category Setting'),
     .....
    'url'       => $this->getUrl('*/*/teamcategory', array('_current' => true)),
    'class'     => 'ajax',
));
```
**SECOND - The method and controller for team category tab** (each tab requires its own Action method) :
```php
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Modulesettings\SettingsController.php
public function teamcategoryAction() {
}    
```

**THIRD - the XML** (each Action method requires xml  nodes):
```html
app\design\adminhtml\default\default\layout\ubt_managestaff.xml
<adminhtml_modulesettings_settings_teamcategory>
    <block type="ubt_managestaff/adminhtml_config_edit_tab_teamcategorybutton" name="teamcategory.form"/>
    <block type="ubt_managestaff/adminhtml_config_grids_teamcategory_grid" name="teamcategory.grid"/>
</adminhtml_modulesettings_settings_teamcategory>

```

**NOTE:**   the first call of the xml is to a custom block. In this example, the block is called 'teamcategory.form'. This block adds a button to the top of the grid page which clicks onto the 'add new entity form'.

The button also stores the **param variables** of the unique grid button being clicked; 
i.e 
```php
//the grid button for the teamcategory 
'onclick' => "setLocation('{$this->getUrl('*/*/editform', array('configtype' => 'teamcategory'))}')", 
```
These params are used in the **editform method** of the  settingsController.php to determine which model should be called and which form block to call i.e: 

```php
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Modulesettings\SettingsController.php
public function editformAction() {
    $configtype = $this->getRequest()->getParam('configtype');
    $model = Mage::getModel("ubt_managestaff/$configtype")->load($id);
    $block = $this->getLayout()
                 ->createBlock("ubt_managestaff/adminhtml_config_grids_edit")
                 ................
                ->_addLeft($this->getLayout()->createBlock("ubt_managestaff/adminhtml_config_grids_{$configtype}_edit_tabs"))
}   
```

**FOURTH - The Forms:**

Each entity within the unique grids has its own form button at the top of the page. Each button leads to a separate form.
Location of the forms:  
```sh
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Config\Edit\Tab
//example of the teamcategory form: 
app\code\local\Ubt\ManageStaff\Block\Adminhtml\Config\Edit\Tab\Teamcategory.php
```

**IMPORTANT NOTES:**

**Delete Button At Top Of Edit Form. ** 

The delete buttons at the top of every edit form are dynamically generated via the **observer pattern** method. The params within the URL of the button are unique to each form; they enable the delete method to determine which model class to load and delete. 


```php
app\code\local\Ubt\ManageStaff\Model\Observer.php

public function addDeletionButton($observer) {
    //these values were set in the editformAction method.
    $configvalues = Mage::registry('moduleConfigValues');
    $data = array('onclick'   => 'setLocation(\'' . $container->getUrl('*/*/delete',  array('configtype' => $configvalues['configtype'],'id'=>$configvalues['id'])) . '\')',
            );
    $container->addButton('my_button_identifier', $data);
}
```
[back to top](#top)</br>
**The delete method:** 

```php
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Modulesettings\SettingsController.php

public function deleteAction(){
    $configModel = $this->getRequest()->getParam('configtype');
    $model = Mage::getModel("ubt_managestaff/$configModel")->load($id);
   $model->delete();
}
```

**SUBMISSION OF FORMS: **
All submitted forms are sent to the same destination   i.e the **save method** of SettingsController:
```php
app\code\local\Ubt\ManageStaff\controllers\Adminhtml\Modulesettings\SettingsController.php

public function saveAction() {
}
```
The **save method** loops through the names of all the tabs to determine which of them set the form. It then load the corresponding named model and saves the post data. 

<a id="frontend-design"></a>
[back to top](#top)</br>

# FRONTEND DESIGN:-

The frontend consists of two pages;

1. Main page: listing of all vacancies.
1. Detailed vacancy page. 


Each 'vacancy listing' on the main page is clickable and leads to in-depth details of the vacancy  page.

<a id="main-page"></a>


### MAIN PAGE:
The **main page phtml** is a composition of several phtml pages; all contained within the folder :

```sh
app\design\frontend\base\default\template\ubt\careers
```
The **controller/method** for the front page is:

```php
app\code\local\Ubt\Careers\controllers\IndexController.php
public function indexAction() {
}  
```

The **XML** for the front page is:

```sh
app\design\frontend\base\default\layout\ubt_managestaff.xml
The node is:  ubt_careers_index_index
```
```xml
<ubt_careers_index_index>
    <reference name="top.container">
        <block type="ubt_careers/vacancy" name="vacancybanner" template="ubt/careers/careers/banner.phtml" />
    </reference>
    <reference name="vacancybanner">
        <block type="core/template" name="teammembertitles" as="vacancy.careers.titles"  template="ubt/careers/careers/banner-title.phtml"/>
    </reference>
    <reference name="content">
        <block type="ubt_careers/vacancy" name="ubt_careers/vacancy.results" template="ubt/careers/careers/list.phtml"/>
    </reference>
    <reference name="manage_staff/teammemberview.results">
        <block type="core/template" name="manage_staff/teammemberview.searchform" as="suppliersearchform"  template="ubt/careers/careers/forms/searchform.phtml"/>
    </reference>
    <reference name="content">
        <block type="ubt_careers/vacancy" name="teammembersbottompage" template="ubt/careers/careers/jqueryscript.phtml" />
    </reference>
</ubt_careers_index_index>
```

- The banner div for the page is located here: **app\design\frontend\base\default\template\ubt\careers\careers\banner.phtml**
-  The 'banner title' (it’s called/activated in the banner.phtml page): **app\design\frontend\base\default\template\ubt\careers\careers\banner-title.phtml**

- The array of vacancy values is loaded here: **app\design\frontend\base\default\template\ubt\careers\careers\list.phtml**

-  The JQuery functionality for the page: **app\design\frontend\base\default\template\ubt\careers\careers\jqueryscript.phtml**
  

<a id="detailed-vacancy-page"></a>

### DETAILED VACANCY PAGE:


The **main page phtml** is a composition of several phtml pages; all contained within the folder:

```sh
app\design\frontend\base\default\template\ubt\careers\careers
```
The **controller/method** for the front page is:

```php
app\code\local\Ubt\Careers\controllers\VacancyController.php
public function indexAction() {
}  
```

The **XML** for the front page is:

```sh
app\design\frontend\base\default\layout\ubt_managestaff.xml
The node is:  ubt_careers_vacancy_view
```
```xml
<ubt_careers_vacancy_view>
    <reference name="top.container">
        <block type="ubt_careers/vacancy" name="vacancybanner" template="ubt/careers/careers/banner.phtml" />
    </reference>
    <reference name="vacancybanner">
        <block type="core/template" name="vacancytitle" as="vacancy.careers.titles"  template="ubt/careers/careers/vacancy-title.phtml"/>
    </reference>
    <reference name="content">
        <block type="ubt_careers/vacancy" name="ubt_careers/vacancy.results" template="ubt/careers/careers/search-result.phtml" />
    </reference>
    <reference name="ubt_careers/vacancy.results">
        <block type="core/template" name="ubt_careers/vacancy.regform" as="vacancyregform"  template="ubt/careers/careers/forms/regform.phtml"/>
    </reference>
    <reference name="content">
        <block type="ubt_careers/vacancy" name="teammembersbottompage" template="ubt/careers/careers/jqueryscript.phtml" />
    </reference>
</ubt_careers_vacancy_view>
```

- The banner div for the page is located here: **app\design\frontend\base\default\template\ubt\careers\careers\banner.phtml**
-  The 'banner title' (it’s called/activated in the banner.phtml page): **app\design\frontend\base\default\template\ubt\careers\careers\vacancy-title.phtml**

- The array values for the clicked vacancy array here: **app\design\frontend\base\default\template\ubt\careers\careers\search-result.phtml**

- The applicant's registration form for the vacancy: **app\design\frontend\base\default\template\ubt\careers\careers\forms\regform.phtml**

-  The JQuery functionality for the page: **app\design\frontend\base\default\template\ubt\careers\careers\jqueryscript.phtml**
 
<a id="future-tasks"></a>
[back to top](#top)</br>

# FUTURE TASK/ IMPROVEMENTS 

- [ ] The front end form submission should be converted to ajax 
- [x] detailed commenting of functions 
- [ ] add PHP unit testing 
- [ ] 





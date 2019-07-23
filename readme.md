# Z Model framework
zModel framework is a predictive model framework built on top of ServiceNow platform. It defines a runtime environment where predictive models can be deployed and used.

Website: https://z-model.com

You build a model outside of ServiceNow using your own machine learning tools and algorithms. Then you use zModel framework to deploy the model into ServiceNow and can use it for any business or IT application and processes.

![img](/assets/img1.png)

![img](/assets/img2.png)

![img](/assets/img3.png)

There are two core entities, which define the framework:
- **Model** - predictive model, which calculates predictive score based on the input data.
- **Score** - result of the calculations for the particular model.

The typical use-case for the framework is the following:
 1. Create / upload the model
 2. Run the model on the particular input data using internal API, REST API or workflows.

The application can also be downloaded from ServiceNow [store](https://store.servicenow.com/sn_appstore_store.do#!/store/application/b2dd32fddb87f240cc38f7fdbf96192f/3.3.1)

## Model
Model defines the logic, which should be used to calculate predictive score for input data.

There are several important fields on the model form:
- **Type** - type of the model, used for information and verification purposes
- **Name** - name of the model, can be defined by a user, used for informational purposes only
- **Description** - description of the model, can be defined by a user, used for informational purposes only
- **Logic** - the most important field, which defines the model coefficients in the particular format. See the corresponding model documentation for the details about the logic format.

>Z Model framework provides three types of the models out of the box:
>
>1. Linear Regression
>2. Logistic Regression (binary classification)
>3. Softmax Regression (multi class)
## Score
Every time when a single score calculated using internal or REST API, Z Model framework creates a record in *Score* table. Each record contains the following fields:
1. Number - record number
2. Transaction reference - external transaction reference, can be useful if you need to store external transaction ID
3. Model - reference to the model
4. User - reference to the user, who runs the transaction
5. Status - status of the transaction, **77 - Success**, **21 - Error**
6. Status details - explicit details about the status
7. Input - input data
8. Score - calculated score / output data

## Internal API

Internal API defined by the script includes, which contains calculation logic for all model types.

#### Use:
```javascript
var zM = new x_elsr_z_model.zModel(); // initialize Z model object
var input_data = "[1,2,3,4,5]"; // input data in a form of stringify array
var model_sys_id = "98340bf1dbc7f240cc38f7fdbf961957"; // sys_id of the corresponding model
var external_reference = "#AEY90"; // optional parameter - external transaction reference, can be any string up to 40 chars

var result = zM.linearModel(input_data, model_sys_id, external_reference);
// the function returns JSON object with the following properties:
// "status" - status of the transaction
// "status details" - status explanation
// "score" - calculated score
// "score sys_id" - sys_id of the created Score record
```
#### Models
Three models provided out of the box:
1. **Linear model** - linear regression model, Score output - stringified JSON matrix with one element type float.
2. **Logistic model** - logistic regression model, Score output - stringified JSON matrix with one element - probability [0..1].
3. **Softmax model** - multi-class logistic regression model, Score output - stringified JSON matrix with probabilities for the corresponding class labels.

##### Use:
```javascript
// linear model
var lm_result = zM.linearModel(input_data, model_sys_id, external_reference);

// logistic model
var lr_result = zM.logisticModel(input_data, model_sys_id, external_reference);

// softmax model
var sm_result = zM.softmaxModel(input_data, model_sys_id, external_reference);
```
## Examples

#### Linear model

```javascript
// initiate new zModel object
var zM = new x_elsr_z_model.zModel();

// input data for a liner regression, 1 - corresponds for a bias
var x = "[96,93,95,1]";

// model sys_id = 98340bf1dbc7f240cc38f7fdbf961957
// external transaction ID = LIN#4735
var result = zM.linearModel(x,"98340bf1dbc7f240cc38f7fdbf961957", "LIN#4735");

gs.log("\n status:"+result.status+"\n status description: "+result['status details']+"\n score: "+result.score+"\n score sys_id: "+result["score sys_id"]);

```
#### Logistic model

```javascript
var zM = new x_elsr_z_model.zModel();
var x = "[1,0,2,1,2,151.55,1]"; // 1 - for the bias
var result = zM.logisticModel(x,"7eb35bf9dbc7f240cc38f7fdbf961948", "LOG#7721");

gs.log("\n\n status:"+result.status+"\n status description: "+result['status details']+"\n score: "+result.score+"\n score sys_id: "+result["score sys_id"]);
```
#### Softmax Regression model

```javascript
var x = "[0.1,0.5,1]"; // 1 - for the bias
var result = zM.softmaxModel(x,"6438137ddbc7f240cc38f7fdbf96195e", "SMX#1217");

gs.log("\n\n status:"+result.status+"\n status description: "+result['status details']+"\n score: "+result.score+"\n score sys_id: "+result["score sys_id"]);
```

## Contacts
Questions, requests for customization or development services - support@dev-labs.io
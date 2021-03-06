# BMI prediction API
## About
This project builds a predictive model based on a training dataset and a server-side API that can take new data and
return a prediction based on the model.

The model aims to predict a person's BMI (body mass index) based on 232 different input variables. Each record in the
datasets used in training/testing the model includes
- ID (index)
- a Body Mass Index or BMI (dependent variable)
- 232 independent variables (including gender, age, two polygenic risk scores, and 228 metabolic biomarkers)

The datasets can be found under the `/data` directory.
 - `dataset1.tsv` (training data)
 - `dataset2.tsv` (test data)

## How to run the program
Using the BMI prediction API consists of 2 steps:
1. Building a Lasso regression model with the `src/model.py` script and 
2. Running the API (`src/api.py`) with Docker.

All the required packages can be installed with `pip install -r requirements.txt`

### Step 1. Build the model
The `model.py` scripts builds a lasso regression model that is trained with the training dataset (`training_data`)
provided. The generated model is saved under the `src` folder with a filename `Lasso_model.pkl`. Optionally, a 
validation of the model can be performed by providing `test_data`. Validation step caclulates the Mean Squared Error (MSE) of
the generated model and draws a plot of the predicted vs. actual BMI values when the model was tested against the test
data set.

Example command:
```
python src/model.py \
    --training_data data/dataset1.tsv \
    --test_data data/dataset2.tsv
```

Arguments:
 * `training_data`: a .tsv file
 * `test_data`: a .tsv file
 * `output_model_name`: string

With the example command above, the script generates the following files:
 * The model file (e.g. `src/Lasso_model.pkl`)
 * (OPTIONAL) A plot (`./plot.png`)

### Step 2: Start the API server
The API was built with [Flask](https://flask.palletsprojects.com/en/2.0.x/quickstart/) and it can be run in a docker
container. To run the API, make sure that the `Lasso_model.pkl` file exists under the `src` directory.  Steps to run the
API in docker container:

**Create a docker image**
```
docker build -t bmi-api:latest .
```
**Run a container**
```
docker run -d -p 5000:5000 bmi-api
```

You can verify that the API is running correctly by listing the currently running containers with `docker ps` command
and by trying to ping the server wit e.g. `curl http://127.0.0.1:5000/ping`, which should return `Ping!`


### Step 3: Use the API to make predictions
The API is documented according to the [OpenAPI Specification]("https://swagger.io/specification/"). The specification
yaml file is named as `openapi.yaml`. One can e.g. copy-paste the contents of this file to https://editor.swagger.io/ to
get an overview of the API.

The API has 2 Endpoints:
* **GET** `http://localhost:5000/ping` (Ping the API server)
* **POST** `http://localhost:5000/bmi` (Predict BMI)

The API's `/bmi` endpoint requires the data (predictor variables) in a JSON format. There is a helper script in
`utils/tsv_to_json.py` that can be used to transform a .tsv file into a JSON file. From that JSON array, one can pick
one of the objects, delete the `BMI` and `ID` features, and use the JSON with remaining features as a request body for
the API call.

Example of making a `POST` request to the API's `/bmi` endpoint:
```
Headers
content-type: application/json

Request body
{"VMALE": "0", "AGE": "36", "BMI32_WGRS": "4.58515", "BMI32_GRS": "32.379", "XXL_VLDL_P": "4.85e-11", "XXL_VLDL_L": "0.0103", "XXL_VLDL_PL": "0.000923", "XXL_VLDL_C": "0.00213", "XXL_VLDL_CE": "0.00162", "XXL_VLDL_FC": "0.000515", "XXL_VLDL_TG": "0.00726", "XL_VLDL_P": "1.11e-10", "XL_VLDL_L": "0.0113", "XL_VLDL_PL": "0.00241", "XL_VLDL_C": "0.00319", "XL_VLDL_CE": "0.00172", "XL_VLDL_FC": "0.00147", "XL_VLDL_TG": "0.00572", "L_VLDL_P": "1.29e-09", "L_VLDL_L": "0.0777", "L_VLDL_PL": "0.017", "L_VLDL_C": "0.0252", "L_VLDL_CE": "0.0175", "L_VLDL_FC": "0.0077", "L_VLDL_TG": "0.0355", "M_VLDL_P": "9.28e-09", "M_VLDL_L": "0.325", "M_VLDL_PL": "0.0748", "M_VLDL_C": "0.128", "M_VLDL_CE": "0.0889", "M_VLDL_FC": "0.039", "M_VLDL_TG": "0.123", "S_VLDL_P": "2.61e-08", "S_VLDL_L": "0.531", "S_VLDL_PL": "0.131", "S_VLDL_C": "0.236", "S_VLDL_CE": "0.155", "S_VLDL_FC": "0.0811", "S_VLDL_TG": "0.164", "XS_VLDL_P": "4.78e-08", "XS_VLDL_L": "0.613", "XS_VLDL_PL": "0.205", "XS_VLDL_C": "0.296", "XS_VLDL_CE": "0.195", "XS_VLDL_FC": "0.102", "XS_VLDL_TG": "0.112", "IDL_P": "1.47e-07", "IDL_L": "1.5", "IDL_PL": "0.414", "IDL_C": "0.945", "IDL_CE": "0.656", "IDL_FC": "0.289", "IDL_TG": "0.144", "L_LDL_P": "2.45e-07", "L_LDL_L": "1.75", "L_LDL_PL": "0.42", "L_LDL_C": "1.2", "L_LDL_CE": "0.857", "L_LDL_FC": "0.338", "L_LDL_TG": "0.135", "M_LDL_P": "1.94e-07", "M_LDL_L": "0.985", "M_LDL_PL": "0.241", "M_LDL_C": "0.677", "M_LDL_CE": "0.502", "M_LDL_FC": "0.175", "M_LDL_TG": "0.0669", "S_LDL_P": "2.11e-07", "S_LDL_L": "0.593", "S_LDL_PL": "0.158", "S_LDL_C": "0.397", "S_LDL_CE": "0.297", "S_LDL_FC": "0.101", "S_LDL_TG": "0.0378", "XL_HDL_P": "3.42e-07", "XL_HDL_L": "0.343", "XL_HDL_PL": "0.199", "XL_HDL_C": "0.129", "XL_HDL_CE": "0.0795", "XL_HDL_FC": "0.049", "XL_HDL_TG": "0.0152", "L_HDL_P": "1.26e-06", "L_HDL_L": "0.785", "L_HDL_PL": "0.41", "L_HDL_C": "0.339", "L_HDL_CE": "0.262", "L_HDL_FC": "0.0768", "L_HDL_TG": "0.0366", "M_HDL_P": "2.4e-06", "M_HDL_L": "1.02", "M_HDL_PL": "0.474", "M_HDL_C": "0.489", "M_HDL_CE": "0.389", "M_HDL_FC": "0.0998", "M_HDL_TG": "0.0558", "S_HDL_P": "5.51e-06", "S_HDL_L": "1.23", "S_HDL_PL": "0.608", "S_HDL_C": "0.567", "S_HDL_CE": "0.445", "S_HDL_FC": "0.122", "S_HDL_TG": "0.0535", "XXL_VLDL_PL_PER": "8.95", "XXL_VLDL_C_PER": "20.7", "XXL_VLDL_CE_PER": "15.7", "XXL_VLDL_FC_PER": "4.99", "XXL_VLDL_TG_PER": "70.4", "XL_VLDL_PL_PER": "21.3", "XL_VLDL_C_PER": "28.2", "XL_VLDL_CE_PER": "15.2", "XL_VLDL_FC_PER": "13", "XL_VLDL_TG_PER": "50.6", "L_VLDL_PL_PER": "21.8", "L_VLDL_C_PER": "32.5", "L_VLDL_CE_PER": "22.6", "L_VLDL_FC_PER": "9.91", "L_VLDL_TG_PER": "45.7", "M_VLDL_PL_PER": "23", "M_VLDL_C_PER": "39.3", "M_VLDL_CE_PER": "27.4", "M_VLDL_FC_PER": "12", "M_VLDL_TG_PER": "37.7", "S_VLDL_PL_PER": "24.7", "S_VLDL_C_PER": "44.4", "S_VLDL_CE_PER": "29.2", "S_VLDL_FC_PER": "15.3", "S_VLDL_TG_PER": "30.9", "XS_VLDL_PL_PER": "33.4", "XS_VLDL_C_PER": "48.3", "XS_VLDL_CE_PER": "31.7", "XS_VLDL_FC_PER": "16.6", "XS_VLDL_TG_PER": "18.3", "IDL_PL_PER": "27.6", "IDL_C_PER": "62.9", "IDL_CE_PER": "43.7", "IDL_FC_PER": "19.2", "IDL_TG_PER": "9.6", "L_LDL_PL_PER": "24", "L_LDL_C_PER": "68.3", "L_LDL_CE_PER": "49", "L_LDL_FC_PER": "19.3", "L_LDL_TG_PER": "7.71", "M_LDL_PL_PER": "24.5", "M_LDL_C_PER": "68.8", "M_LDL_CE_PER": "51", "M_LDL_FC_PER": "17.8", "M_LDL_TG_PER": "6.79", "S_LDL_PL_PER": "26.7", "S_LDL_C_PER": "66.9", "S_LDL_CE_PER": "50", "S_LDL_FC_PER": "16.9", "S_LDL_TG_PER": "6.37", "XL_HDL_PL_PER": "58.1", "XL_HDL_C_PER": "37.5", "XL_HDL_CE_PER": "23.2", "XL_HDL_FC_PER": "14.3", "XL_HDL_TG_PER": "4.44", "L_HDL_PL_PER": "52.2", "L_HDL_C_PER": "43.2", "L_HDL_CE_PER": "33.4", "L_HDL_FC_PER": "9.79", "L_HDL_TG_PER": "4.67", "M_HDL_PL_PER": "46.5", "M_HDL_C_PER": "48", "M_HDL_CE_PER": "38.2", "M_HDL_FC_PER": "9.81", "M_HDL_TG_PER": "5.48", "S_HDL_PL_PER": "49.5", "S_HDL_C_PER": "46.2", "S_HDL_CE_PER": "36.2", "S_HDL_FC_PER": "9.94", "S_HDL_TG_PER": "4.35", "VLDL_D": "34.9", "LDL_D": "23.7", "HDL_D": "9.91", "SERUM_C": "5.43", "VLDL_C": "0.691", "REMNANT_C": "1.64", "LDL_C": "2.27", "HDL_C": "1.52", "HDL2_C": "1.02", "HDL3_C": "0.504", "ESTC": "3.76", "FREEC": "1.67", "SERUM_TG": "0.992", "VLDL_TG": "0.447", "LDL_TG": "0.24", "HDL_TG": "0.161", "TOTPG": "2.3", "TG.PG": "0.439", "PC": "2.27", "SM": "0.53", "TOTCHO": "2.77", "APOA1": "1.6", "APOB": "0.989", "APOB.APOA1": "0.619", "TOTFA": "12.6", "UNSAT": "1.11", "DHA": "0.128", "LA": "3.32", "FAW3": "0.379", "FAW6": "3.97", "PUFA": "4.35", "MUFA": "3.22", "SFA": "5.07", "DHA.FA": "1.01", "LA.FA": "26.3", "FAW3.FA": "3", "FAW6.FA": "31.4", "PUFA.FA": "34.4", "MUFA.FA": "25.5", "SFA.FA": "40.1", "GLC": "4.14", "LAC": "1.92", "PYR": "0.125", "CIT": "0.0963", "GLOL": "0.0722", "ALA": "0.438", "GLN": "0.584", "GLY": "0.429", "HIS": "0.0937", "ILE": "0.0469", "LEU": "0.07", "VAL": "0.171", "PHE": "0.0775", "TYR": "0.0572", "ACE": "0.0588", "ACACE": "0.0754", "BOHBUT": "0.103", "CREA": "0.0712", "ALB": "0.102", "GP": "1.16"}
```

To stop the API, simply run
```
docker stop <CONTAINER_ID>
```
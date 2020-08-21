 <header>
Working with the FHIR Resource Repository
=========================================
</header>
<main>

# Introduction

InterSystems IRIS for Health™ provides a base FHIR server implementation with an internal FHIR resource repository for storing and managing FHIR resources. In this lab you will set up a FHIR R4 server with a resource repository, populate the repository with realistic sample patient data and construct a simple FHIR client app that can interact with the server via FHIR RESTful API.

# Goals

By the end of this exercise, you should be able to:

*   Set up a FHIR resource repository using InterSystems IRIS for Health™
*   Use the open source Synthea™ tool to generate synthetic patient data
*   Use a REST client to manually interact with the FHIR server via FHIR RESTful API
*   Set up a simple web-based FHIR app to retrieve and display data from the FHIR repository

> ---
> This exercise requires the use of a web browser to access certain webpages that no longer support Internet Explorer. We strongly recommend using **Google Chrome** for the exercise, though Mozilla Firefox should also work.
> 
> ---

# Instructions

## Set Up a FHIR R4 Server with a Resource Repository

InterSystems IRIS for Health includes an installer method for creating a FHIR server namespace with a built-in resource repository. For our exercise we will make use of this method to create a brand new namespace called FHIRSERVER and we will set up it as a FHIR R4 server.

1.  Select **InterSystems > Web Terminal** from the dropdown menu at the top of the IDE to open InterSystems Terminal. Supply **tech/demo** for credentials when asked. Run the following command to create a new IRIS for Health namespace called `FHIRServer`:

    &nbsp;&nbsp;&nbsp;`do ##class(HS.HC.Util.Installer).InstallFoundation("FHIRServer")`

2.  When the command finishes running, from the IDE, open Management Portal by selecting **InterSystems > IRIS Management Portal** from the dropdown menu at the top. Terminal will be used again in the next section, so please do not close the Terminal window. Supply **tech/demo** for credentials in the Management Portal.

3. From the Management Portal homepage, switch to the FHIRSERVER namespace and navigate to **Health** > **FHIR Configuration** > **Server Configuration** to set up a new FHIR R4 endpoint that stores FHIR data as JSON in a FHIR resource repository. Modify the following settings:

    -  **Metadata** : HL7v40
    - **Interaction strategy**: _HS.FHIRServer.Storage.Json.InteractionsStrategy_
    - **URL** : /_csp/healthshare/fhirserver/fhir/r4_

4. Click the /csp/healthshare/fhirserver/fhir/r4 endpoint, click **Edit**, and then select **Allow Unauthenticated Access**.

## Populate the Repository with Synthetic Patient Data

In this section, we will populate FHIRSERVER's resource repository with realistic patient data, generated using Synthea™ Patient Generator. Synthea produces JSON files, each being a FHIR Bundle containing various FHIR resources for a single patient, such as Patient, Condition, Medication and Observation.

> ---
> Synthea™ is an open-source patient generator that models the medical history of synthetic patients, providing high-quality, synthetic, realistic but not real, patient data and associated health records covering various aspects of healthcare. For more information, please see: [https://synthetichealth.github.io/synthea](https://synthetichealth.github.io/synthea)
>
> ---

15 synthetic patients (15 JSON files) have already been pre-generated for this exercise, and we will use these samples to populate the FHIR R4 resource repository of FHIRSERVER. If time allows, as an optional step at the end of this exercise, you can try installing Synthea Patient Generator on your computer and generate your own synthetic patients. For loading data into the resource repository, IRIS for Health provides a utility method that allows you to directly submit multiple FHIR bundles just by specifying the location of a directory where sample files are found.

1.  Locate the pre-generated FHIR bundles (JSON files) under the **/shared/FHIRLab/samples** directory in the server container, accessible through the integrated IDE.


2.  Expand the **samples** folder and double-click the first sample file (i.e., **Allen_Runte_9e43a3bf-fb4f-4007-8a1f-d8e00e57d4e5.json**) to see what a bundle of FHIR resources looks like in JSON.

3.  From **Web Terminal**, switch to the FHIRSERVER namespace that was created in the previous section.

    &nbsp;&nbsp;&nbsp;`HSLIB> set namespace = "FHIRSERVER"`

4.  Execute the following command, making sure to specify the correct input directory location, as shown below. The command may take a while to load all 25 bundles, which are relatively large in size.

    &nbsp;&nbsp;&nbsp;`FHIRSERVER> set sc = ##class(HS.FHIRServer.Tools.DataLoader).SubmitResourceFiles("/home/project/shared/FHIRLab/samples","FHIRServer","/csp/healthshare/fhirserver/fhir/r4")`

5.  When the command finishes running, check the console to ensure that all 15 synthetic patients have been loaded successfully.

## Use a REST Client to Search and Update Patient Data in FHIR Repository

In this section, we will use a REST client to manually interact with FHIRSERVER via FHIR RESTful API. InterSystems IRIS for Health supports all client/server interactions defined under the FHIR specification, including create, read, update, delete, patch, and search. Let's start by creating a new Observation resource for a patient whose name is Abbie Ortiz.

### Search for Patient data in FHIR Resource Repository

1. First, open your REST Client to perform a search for Ciara Jenkins's patient resource by completing the fields below with the following values:

**Headers** :

- *Accept*: *application/fhir+json*
- *Content-type*: *application/fhir+json*

**HTTP Request** : `GET http://localhost:52773/csp/healthshare/fhirserver/fhir/r4/Patient?family=Jenkins&given=Ciara`

2. Submit the request and examine the response payload bearing Ciara Jenkins's Patient resource in a FHIR bundle and confirm that her ID is 1158.

3. You can get any other information about her, including related resources such as AllergyIntolerance and Encounter, by adding `$everything` to the end of your URL:

`http://localhost:52773/csp/healthshare/fhirserver/fhir/R4/Patient/1158/$everything`

### Update patient data in FHIR Resource Repository

1. Update Ciara's data by creating a new Observation resource for her using the provided sample Observation input, corresponding to Ciara's BMI obtained from her recent test by completing the following steps:

  1. Change HTTP Request to `POST http://localhost:52773/csp/healthshare/fhirserver/fhir/R4/Observation`
  2. In the file explorer panel of your IDE, locate and open **NewObservation-Ciara.json** under the **/etc** directory.
  3. Make note of the LOINC code (39156-5) corresponding to BMI, as well the observation date (effectiveDateTime = **2020-08-15T10:55:05-04:00** ) and the BMI value (valueQuantity.value = **26.85** ) indicated in the sample input.
  4. Copy the entire JSON content of this sample input file and paste into the **Body** section of the REST Client window (under the Body tab of Chrome's Advanced REST Client).
  5. Submit the request and verify that a **201 Created** response is returned from the server, indicating a successful creation and storage of the new Observation resource.

2. To confirm resource creation, perform a search to retrieve the newly added Observation resource, by change the HTTP Request to

`GET http://localhost:52773/csp/healthshare/fhirserver/fhir/R4/Patient/1158/Observation?code=39156-5&date=2020-08-15`

## Construct a Simple FHIR Client App

In this section we will connect a sample FHIR client app to our FHIR server/repository and see the app display interesting patient data on a webpage. This sample app allows a user to identify diabetes patients and plot their BMI on a chart.

1.  Let's start by opening the sample app file in the IDE (**/shared/FHIRLab/app/FHIRResourceRepository.html**).

> ---
> The sample FHIR app (FHIRResourceRepository.html) is a simple JavaScript-based webpage that internally performs the following two queries against the FHIR server:
>
> https://hostname:port/csp/healthshare/fhirserver/fhir/R4/Patient?**_has:Condition:patient:code**=**15777000&birthdate=lt1955-01-01&_sort=-birthdate&_count=7**
>*  This query returns "7 oldest diabetes patients older than 65."
>*  Here **code=15777000** refers to the SNOMED-CT code for **Diabetes**.
>*  This search uses the **_has** parameter, allowing one to perform a search with reverse chaining. In this case, the search is looking for any patient who has at least one Condition resource whose code corresponds to Anemia. For more information on chained parameters and reverse chaining, please see [https://www.hl7.org/fhir/search.html#has](https://www.hl7.org/fhir/search.html#has).
>*  The **birthdate=birthdate=lt1955-01-01** parameter uses a comparison operator **lt** (less than) to find patients older than 65 (born in or before 1955). Other comparison operators are **le** (less than or equal), **gt** (greater than) and **ge** (greater or equal).
>*   **_sort** is a standard parameter for sorting, with the optional "-" prefix indicating decreasing order.
>*   **_count** is astandard parameter for specifying how many results are to be returned by the FHIR server in a single page.
>
>  https://hostname:port/csp/healthshare/fhirserver/fhir/R4/**Patient/x/Observation?code=39156-5&_sort=date**
>*  This query returns "all BMI observations for patient x, sorted by observation date in increasing order."
>*  As mentioned in a previous section, **code=39156-5** refers to the LOINC code for BMI [kg/m2].
> ---

> ---
> Please note that the sample app makes direct use of the following two free, open source JavaScript libraries, plus some custom JavaScript code to build out the webpage:
> *   **jqFhir.js**: This is a [jQuery](https://jquery.com/)-based implementation of the **fhir.js** library, which provides a simple FHIR client module for performing basic CRUD (create, read, update and delete) operations. Please see [https://github.com/FHIR/fhir.js](https://github.com/FHIR/fhir.js) for more information.
> *   **plotly-latest.min.js**: Plotly ([https://plot.ly/javascript/](https://plot.ly/javascript/)) is a data visualization library, which our sample app uses to graph patient hemoglobin data on a chart.
> ---

2. Modify the **baseUrl** parameter in the the line 44 to make the client point to your FHIR server's FHIR R4 endpoint, by setting baseUrl to 'http://localhost:52773/csp/healthshare/fhirserver/fhir/R4'

3. Scroll down to the **client.search()**, which performs a search for pre-diabetes patients. Follow the instruction to uncomment the line and line to narrow the search down to diabetes patients older than 65 years old.

```
client.search({
 type:'Patient',
 query: {
 // Uncomment the following two lines
 // '\_has:Condition:patient:code':'15777000',

// birthdate='lt1955-01-01',
 \_sort:'-birthdate',
 \_count: 7
 }})
 ```

4. Open the FHIR App web page by right clicking on **/shared/FHIRLab/app/FHIRResourceRepo.html** and select **Open With** > **Preview**. Notice in the URL that this is the page you were just editing, and it displays a list of seven patients. Supply the ID for Elias Barrows to his BMI observation values charted. How is Elias doing?

5. (Optional) Challenge: Glucose level is an important factor for diabetes patients as well. Modify FHIRResourceRepo.html to get Elias's observation values of glucose level (LOINC 2339-0) charted. Note that the value range for oxygen saturation is 60–100 mm/dL. See FHIRResourceRepoSolution.html for the answer.

## (Bonus) Generate Your Own Synthetic Patients

In this section you will download and install Synthea Patient Generator on your computer and generate your own synthetic patients as FHIR R4 resources.

> ---
> Synthea Patient Generator requires Java (JDK) 8.0 or higher installed on your computer. If not already installed, please install Java ([https://www.oracle.com/technetwork/java/javase/downloads/index.html](https://www.oracle.com/technetwork/java/javase/downloads/index.html)) before proceeding further.
> 
> ---

1. Clone Synthea's GitHub page: [https://github.com/synthetichealth/synthea](https://github.com/synthetichealth/synthea)
2. Unzip synthea-master.zip into a local directory, which creates a folder called **synthea-master**.
3. Open **synthea.properties** found under **/synthea-master/src/main/resources**.
4. Set following properties so that the tool can generate FHIR R4 resources (the latest version of Synthea produces FHIR R4 resources by default). Then, click Save.

    - `exporter.fhir\_R4.export = true`
    - `generate.append\_numbers\_to\_person\_names = false`

	
    > ---
    > Setting **exporter.csv.export** to _true_ instructs the tool to produce CSV files corresponding to the FHIR resources generated. This can be useful because it allows users to view data for all patients in a table format.
	> 
    > Setting **generate.append_numbers_to_person_names** to _false_ disables the default behavior of adding random numbers to patient names. Removing the numbers makes the synthetic data more realistic.
	>
    > ---

5. Open Terminal on your computer and navigate to the **synthea** directory you recently cloned and execute the following command:

    `./run\_synthea`

6. Using your computer's file browser, navigate to the **/synthea-master/output/fhir** directory, now containing JSON files for the patient data just generated.

7. Using the IDE, create a new **test** folder by selecting the FHIRLab folder and then going to **File** > **New Folder**. Then, copy the generated files and make them available on the server container by dragging all the JSON files under **/synthea-master/output/fhir** into the new **test** folder.

8. Submit your sample files into FHIRSERVER by repeating relevant steps in the section Populate the Repository with Synthetic Patient Data, starting at Step 3. Make sure to set the source directory to **/home/project/shared/FHIRLab/test**

9. Try accessing your new patients by performing a query from a REST client or by further tweaking the internal queries of the sample FHIR app.

## Summary

In this exercise we used InterSystems IRIS for Health to set up a FHIR server with an internal FHIR repository and loaded the repository with synthetic patients. You then were able to see how the server interacts with client applications through a REST client manually and through a simple FHIR app. Feel free to continue to experiment with the FHIR resource repository by adding more data, or by setting up your own application.
</main>

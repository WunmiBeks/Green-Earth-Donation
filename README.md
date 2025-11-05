# GreenEarth Personalized Thank-You Email (AMPscript Project)

<div align="center">

<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Images/GreenEarthThumbnail.jpeg" width="600" alt="GreenEarth Hero Image"/>

*Personalized thank-you email built in Salesforce Marketing Cloud using AMPscript, HTML, and CSS.*

</div>

---


## Project Overview

## Project Overview

A dynamic AMPscript-driven thank-you email created for **GreenEarth** to recognize donors based on total contributions.  
The project demonstrates **personalization without Journey Builder**, using AMPscript to calculate total donations, render conditional content, and display dynamic images hosted in **Salesforce Content Builder** based on each donor’s supported cause.

---

## Solution Walkthrough


## Solution Walkthrough
This walkthrough outlines the data setup, AMPscript logic, and email previews used to demonstrate personalization in Salesforce Marketing Cloud.

### Step 1: Data Extensions Setup

The project relies on three related Data Extensions that manage donor information, track donations, and reference supported environmental projects.

<div align="center">
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Images/SendingDE.png" width="700" alt="SendingDE Data Extension Screenshot"/>
<br>
*Screenshot — `SendingDE` Data Extension storing donor details and personalization fields.*
</div>

<div align="center">
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Images/Donations.png" width="700" alt="Donations Data Extension Screenshot"/>
<br>
*Screenshot — `Donations` Data Extension logging individual donations for each donor.*
</div>

<div align="center">
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Images/Projects_Data.png" width="700" alt="Projects_Data Data Extension Screenshot"/>
<br>
*Screenshot — `Projects_Data` Data Extension listing available environmental projects for lookup and display.*
</div>


---

### Step 2: AMPscript Logic

```
%%[

/* RETRIEVE FIELDS */
SET @fname = [FirstName]
SET @lname = [LastName]
SET @donorID = [DonorID]
SET @impactArea = [ImpactArea]


/* RETRIEVE PROJECT DATA */
SET @projectName = Lookup("Projects_Data", "ProjectName", "ImpactArea", @impactArea)
SET @projectStatus = Lookup("Projects_Data", "CompletionStatus", "ImpactArea", @impactArea)
SET @projectDesc = Lookup("Projects_Data", "ProjectDescription", "ImpactArea", @impactArea)
SET @projectCompDate = Lookup("Projects_Data", "CompletionDate", "ImpactArea", @impactArea)

SET @projectCompDateFormatted = FormatDate(@projectCompDate, "MMMM, YYYY")

/* FNAME AND LNAME CLEANUP */
IF EMPTY(@fname) THEN
  SET @finalName = "Valued Customer" 
  SET @subjectLine = "Thank You! Here's How Your Support is Making an Impact"
ELSE
  SET @finalFirstName = Propercase(Trim(@fname)) 
  SET @subjectLine = CONCAT("Thank You, ", @finalFirstName, "! Here’s How Your Support is Making an Impact")
  IF EMPTY(@lname) THEN
    SET @finalName = @finalFirstName
  ELSE
    SET @finalLastName = Propercase(Trim(@lname))
    SET @finalName = CONCAT(@finalFirstName, " ", @finalLastName)
  ENDIF
ENDIF

/* LOGIC TO CALCULATE TOTAL DONATION AMOUNT */
SET @donationTotal = 0

SET @rows = LookupRows("Donations","DonorID", @donorID)
SET @rowCount = rowcount(@rows)

IF  @rowCount > 0 THEN

  for @i = 1 to @rowCount do

    SET @row = row(@rows, @i) /* get row based on counter */
    SET @DonationAmount = field(@row,"DonationAmount")
    SET @donationTotal = Add(@donationTotal, @DonationAmount)
    
  next @i 

ELSE
  RaiseError("No donations found")
ENDIF

/* LOGIC FOR DYNAMIC DONATION COPY */
IF @donationTotal < 500 THEN
  SET @donationCopy = "We appreciate your support. Every bit of your contribution counts."

ELSEIF @donationTotal >= 500 AND @donationTotal < 1000 THEN
  SET @donationCopy = "As a token of our appreciation, we would like to invite you to an exclusive donor event."
  SET @URL = "https://www.greenearth.org/learn-more"
  SET @linkText = "Click here to learn more"

ELSEIF @donationTotal >= 1000 AND @donationTotal < 5000 THEN
  SET @donationCopy = "We have a special thank-you video message from our CEO just for you."
  SET @URL = "https://www.greenearth.org/thankyou-video"
  SET @linkText = "Watch it here"

ELSE
  SET @donationCopy = "We are honored to offer you a private tour of one of our conservation sites."
  SET @URL = "https://www.greenearth.org/schedule-tour"
  SET @linkText = "Schedule your visit here"
  

ENDIF

/* FORMAT TOTAL DONATION AMOUNT*/
SET @totalContributions = FormatNumber(@donationTotal, "C")

/* DYNAMIC IMAGE */
IF @impactArea == "Wildlife" THEN 
  SET @image = "http://image.greenearth.ca/lib/fe2f117371640475751078/m/1/5c96e4c6-b449-4d8a-b9c9-b8d84b845f1e.jpg"

ELSEIF @projecimpactAreatName == "Water" THEN 
  SET @image = "http://image.greenearth.ca/lib/fe2f117371640475751078/m/1/238749be-1ad1-4af5-b6a6-90b6fa44db95.jpeg"

ELSEIF @impactArea == "Energy" THEN
  SET @image = "http://image.greenearth.ca/lib/fe2f117371640475751078/m/1/735f1ad4-1f2d-4083-8d26-c8f5d4435037.jpg"

ELSE 
  SET @image = "http://image.greenearth.ca/lib/fe2f117371640475751078/m/1/b01ee012-5327-42ac-854b-e713dd8d7f50.jpg"

ENDIF

]%%
```
<div align="center">
<em>AMPscript logic block — retrieves donor data, calculates total donations, and renders personalized thank-you content.</em>
</div>

---

### Step 3: Email Preview & Testing

This section demonstrates how the personalized thank-you email renders dynamically using data from multiple Data Extensions. The previews below show how different fields and personalization logic connect to the donor, donation, and project data.

---

#### 1. Email Template with AMPscript Personalization Strings

<div align="center">
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/ProjectScreenshots/emailPreview_Sample2.png" width="750" alt="Email Template with Personalization Strings"/>
<br>
<em>Initial email template showing personalization strings used to render dynamic content.</em>
</div>

---

#### 2. Preview Comparison – Donor Data 

<div align="center">
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Projects_Data/dynamicEmailPreview1.png" width="750" alt="Email Preview with Donor Data"/>
<br>
<em>Preview showing how donor attributes (First Name, Last Name, Impact Area, Tier) from the Sending Data Extension populate the email dynamically.</em>
</div>

---

#### 3. Preview Comparison – Donations Data 

<div align="center">
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Projects_Data/dynamicEmailPreview2.png" width="750" alt="Email Preview with Donation Data"/>
<br>
<em>The donation amount displayed in the preview is dynamically pulled from the <strong>DonationAmount</strong> field in the <strong>Donations</strong> Data Extension.</em>
</div>

---

#### 4. Preview Comparison – Project Data 

<div align="center">
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Projects_Data/EmailPreview_ProjectData.png" width="750" alt="Email Preview with Project Data"/>
<br>
<em>Highlighted fields such as <strong>Project Name</strong>, <strong>Description</strong>, <strong>Completion Status</strong>, and <strong>Completion Date</strong> are dynamically pulled from the <strong>Projects_Data</strong> Data Extension.</em>
</div>

---

#### 5. Dynamic Email Previews for Multiple Donors

<div align="center">
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Images/dynamicEmailPreview1.png" width="750" alt="Dynamic Email Preview 1"/>
<br><br>
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Images/dynamicEmailPreview2.png" width="750" alt="Dynamic Email Preview 2"/>
<br><br>
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Images/dynamicEmailPreview3.png" width="750" alt="Dynamic Email Preview 3"/>
<br>
<em>Final test emails showing dynamic images and personalized content rendered for different donors based on their data.</em>
</div>

#### 6. Final Email Output

Below are two examples of the final thank-you email as received by different donors after AMPscript rendering and personalization.  
Each version dynamically displays the donor’s name, total contributions, project information, and an image corresponding to their selected impact area, retrieved from **Salesforce Content Builder**.

<div align="center">
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Images/TestEmail_Received_1.png" width="750" alt="Final Test Email – Wildlife Donor"/>
<br>
<em>Final rendered email for a donor supporting the <strong>Wildlife</strong> project — personalized copy, donor data, and corresponding image displayed.</em>
</div>

<br>

<div align="center">
<img src="https://github.com/WunmiBeks/Green-Earth-Donation/raw/main/Images/TestEmail_Received_2.png" width="750" alt="Final Test Email – Energy Donor"/>
<br>
<em>Final rendered email for a donor supporting the <strong>Energy</strong> project — dynamic content and visuals reflecting the donor’s selected cause.</em>
</div>

---

## Project Summary

This project showcases how AMPscript can be used to create dynamic, data-driven personalization in Salesforce Marketing Cloud.  
Built entirely with AMPscript, it demonstrates how data from multiple extensions powers targeted and personalized donor communications directly within the email template.

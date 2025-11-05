# GreenEarth Personalized Thank-You Email

## Project Overview

This project involves creating a personalized thank-you email for **GreenEarth** using **Salesforce Marketing Cloud (SFMC)**. The email dynamically adjusts content based on the donor's contribution amount, an impact report highlighting relevant projects supported by the donor's funds.It also displays personalized thank-you messags based on amount donated

---

## Scenario

GreenEarth aims to deliver a personalized thank-you email that showcases the real impact of each donor’s contributions through a detailed project report. The email dynamically adapts based on the donor’s total contribution amount, offering customized messages and exclusive perks for higher-tier donors. The total donation is calculated by aggregating multiple records from a separate data extension.


## Requirementsnumber 

### Personalization

- Greet the donor by their **first name**.
- Calculate the total donation amount by summing records from a separate data extension.
- Include the **total donation amount** in the email.
- Dynamically insert a **personalized thank-you message** based on the amount donated by the donors.


### **Conditional Messaging**
| **Tier** | **Reward** |
|-----------|-------------|
| $500+ | Invitation to exclusive event |
| $1,000+ | Thank-you video from CEO |
| $5,000+ | Private tour of a conservation site |


### **Data Transformation**
- Properly capitalize names  
- Format donation amounts and event dates  

### **Dynamic Content**
- Adjust email copy dynamically based on donor tier  
- Display real-time project completion percentage and status


## Data Extensions

### Sending Data Extension
<Image SendingData>
*Data Extension that stores donors' data*


### **Donations**

<Image Donations>
*Data Extensions that stores donation records 


### **Projects_Data**

<Image Projects_Data>
*Data Extension that stores project records*


##  Dynamic Message Table
| **Donation Range** | **Message** |
|---------------------|-------------|
| < $500 | We appreciate your support. Every bit of your contribution counts. |
| $500 – $999 | As a token of our appreciation, we invite you to an exclusive donor event. |
| $1,000 – $4,999 | We have a special thank-you video message from our CEO just for you. |
| ≥ $5,000 | We’re honored to offer you a private tour of one of our conservation sites. |

---

## 💻 Email Logic (AMPscript)

```ampscript
/* RETRIEVE FIELDS */
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





Email Template

Subject Line: %%=v(@subjectLine)=%%

Dear %%=v(@finalName)=%%,

We are deeply grateful for your generous contribution of %%=v(@totalContributions)=%% to GreenEarth. Your support is making a tremendous difference in the area of %%=v(@impactArea)=%%.

Project Update:
You'll be pleased to know that the %%=v(@projectName)=%% project is currently %%=v(@projectStatus)=%% complete. This project is focused on %%=v(@projectDesc)=%% and is expected to be completed by %%=v(@projectCompDateFormatted)=%%.

%%=v(@donationCopy)=%%

A Special Message from Our CEO:

Dear %%=v(@finalName)=%%,
Your commitment to our mission at GreenEarth is truly inspiring. Together, we are making the world a better place. Thank you for being a vital part of our community.

Warm regards,
CEO Name

Thank you once again for your support.

We look forward to continuing this journey with you and making even greater strides in our mission.



### Email Preview

*Preview and Test in SFMC*

<Email Preview>


*Previewed against the sending DE*
<Email Image>


*Previewed against the Project Data DE*
<Email Image>


*Previewed against the Donations DE*
<Email Image>


*Email previwed showing summed up donation amount 


## Test Email 

<Image Email>

# AltPartNumDatalinkECM
Update Line Item PartNums in SO Automation to an Alternate(Substitute) PartNum if there is one available.

## Overview
Within Kinetic there are numerous types of Alternative Part Numbers.  This guide is referring to the PartSubs table found directly within Part Entry under the Alternates pane. These are Substitute Parts, not to be confused with Customer Part Cross Reference parts, or Supplier Part Cross Reference Parts. 

![](images/00-KineticAlternateSubstituteParts.png)

## Creating The Datalink

Add the following Datalink to your CallDefinitions.json file or add it using the JSON Web Service built directly into ECM.     

Update ENVIRONMENTURL to match your Environment.  

```
{
    "Name": "AlternatePart",
    "Url": "ENVIRONMENTURL/api/v1/Erp.BO.AlternatePartSvc/AlternateParts?%24filter=Company%20eq%20'@Company'%20and%20PartNum%20eq%20'@PartNum'%20and%20RecType%20eq%20'S'",
    "Method": "Get",
    "AuthScheme": "@TokenType",
    "AuthParameter": "@AccessToken",
    "AllowInvalidSSL": false,
    "CallChain": "ErpLogin",
    "LogPath": "C:\\temp\\AlternatePart.log",
    "WSHeaders": [
      {
        "Key": "CallSettings",
        "Value": "{Company: \"@Company\", PartNum: \"@PartNum\", RecType: \"@RecType\"}"
      }
    ],
    "CollectionPath": "value",
    "Columns": [
      {
        "Name": "Company",
        "Path": "Company"
      },
      {
        "Name": "PartNum",
        "Path": "PartNum"
      },
      {
        "Name": "RecType",
        "Path": "RecType"
      },
      {
        "Name": "SubPart",
        "Path": "SubPart"
      }
    ]
  }
```

This datalink uses the Rest Call, AlternatePartSvc, to find if the PartNum has a matching Substitute Part within Kinetic. 

![](images/15-RestAPI.png)

## Creating Fields and Field Groups

Within the workflow we'll need to store the datalink's results in a fieldgroup.  We'll create the field group in ECM and call it **SUBPART**.

Add two fields to the fieldgroup called: 
- SUBPART_PartNum
- SUBPART_AltPartNum

![](images/20-FieldsAndFieldGroup.png)

## Adding an Additional Step to the Workflow

Below is the stock SO Automation Workflow. 

![](images/05-StockSOWorkflow.png)

We'll an additional step to the Stock Workflow after **Check Line Items**.  Update the branching accordingly so that the steps flow properly.   

![](images/10-AltPartWorkflow.png)

## Adding an Action to the New Step

Within our new step we will add a single **Action**. You can call it *Update Part Num*. We will add four tasks to that Action:
- Datalink Field Group
- Aggregrate
- End Action
- Advanced Math

![](images/25-AltPartNumTasks.png)

## Adding the Tasks to the Action

The first task we'll add is the DataLink Field Group task. It runs the datalink against the field group **LineItems**.  The output is stored in the field group we created called SUBPARTS. 

![](images/30-DatalinkFieldGroupToTempTable.png)

The second task we'll add is an Aggregate.  It will perform the operation **ANY**.  This will determine how many lines on the sales order had substitute parts.  The total value is stored in a temporary field, zCountSubParts. 

![](images/35-AggregateCheck.png)

The third task we'll add is an End Action.  This references the temporary field, zCountSubParts. We'll check the inverse.  If zCountSubParts equals zero, end the action.  

![](images/40-EndActionCheckAggregate.png)

The fourth task we'll add is an AdvancedMath task.  AdvancedMath is capable of updating a field group without entirely resetting the field group.  We will use this capability to individually select specific lines and update those lines if they have a substitute part. 

We'll use the following expression to only update lines where we found a substitute part.   

```
IIF($FieldGroup.SubPart.SUBPART_PartNum = $Field.LINE_PartNum,$FieldGroup.SubPart.SUBPART_AltPartNum,$Field.LINE_PartNum)
```

![](images/45-AdvancedMathUpdate.png)





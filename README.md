# AltPartNumDatalinkECM
Update Line Item PartNums in SO Automation to an Alternate PartNum if there is one available.

## Overview
Within Kinetic there are numerous types of Alternative Part Numbers.  This guide is referring to the PartSubs table found directly within Part Entry under the Alternates pane. These are Substitute Parts, not to be confused with Customer Part Cross Reference parts, or Supplier Part Cross Reference Parts. 

![](images/00-KineticAlternateSubstituteParts.png)

## Changes to the Stock Workflow

![](images/05-StockSOWorkflow.png)

We'll an additional step to the Stock Workflow after **Check Line Items**. 

![](images/10-AltPartWorkflow.png)

## The Datalink

## Creating Fields and Field Groups





## mePrism

## Data Score Journal

### Summary

This document is the journal of the journey required to development and deploy the mePrism Data Score. This project is described by the note from Tom below:

```
-Move the training data set (zip file to be provided) to our company platform in Amazon for current and future data science and analytics. 
-Make sure that this data is clean, organized and usable. This set includes over 1000 people who have submitted their Google Takeout/FB files or Instagram files, completed at least a 50 question Ocean survey and filled out a demographic survey. It is a lot of information. 
-Check Jinyi's work to make sure we are not replicating existing work, some of this training set has been used to build things like the personality (OCEAN) prediction model. 

Build the Data Score.
The data score is intended to be a measure of the value of the user's virtual footprint by assessing the accuracy and completeness of the person's virtual footprint data. 

-We can measure the person's personality based on Jinyi's model.
-We can measure the kinds of demographic data that are specific to each person
-we can measure someone's political leanings from the content of search
-We can measure the volume of data in the person's VP
-We have spending data through plaid (but not in training set)
-We have location history in google takeout
-We have live location data from the phone (but not in training set)
```

### Getting Started

The first point is acquiring the training data set and putting it in a Data Lake on AWS. For this purpose, Tom has sent a link via email to a .zip file with the data.

#### Interruption number one

My work is more than just data science. CapTech has added a resource, Mary Kwiatkowski, to the project. I have provided her accounts for Jira, Slack, GitHub, and AWS. 

**Jira** In Jira's case, I opened my bookmark for the FSA board. There is a *waffle* in  the upper left corner that contains an option for Administration. The Administration link opens the *Users* page. On the Users page there is a button to *Invite Users*. This action will allow you to invite users, assign them a role, and a put them on a team.

**Slack** In the upper left corner of the Slack app (desktop version) there is a *mePrism* menu. That menu contains an item to *Invite people to mePrism*. That action displays another menu to choose the type of user: Members, Multi-channel Guests, and Single-channel Guests. For CapTech I choose Members. That link shows a form where the new user's email address is entered.

**GitHub** Start my opening the organization page for mePrism. I get to this from a link in *Organizations* on my home page. The direct link is https://github.com/mePrism-Inc. From the organization page, click the people tab. There will be an *Invite member* button on the subsequent page.

**AWS** Log onto the AWS Console and select the IAM service. In the left panel menu, click **Users**. There is an *Add User* button on the subsequent page. Type a user id for AWS; I have adopted an informal standard of firstname.lastname. For CapTech, since they will be writing CI/CD scripts, be sure to check both *Programmatic Access* and *AWS Management Console access*. Continue clicking through the wizard. I suggest setting the initial password. I also suggest forcing a password reset on the first login. And lastly, I suggest letting AWS format the first email to the new users.

**FireBase** Start the Firebase console. From the home dashboard, select the card for a project. In this case is it *mePrism-dev-staging*. On the left side menu, choose the gear icon on *Project Overview*, then select Users and Permissions. There is a button for *Add User* that where an email address is entered and permissions granted.

#### Back to the data

Tom's link did not provide access. Instead I received a 403 (forbidden) status. I tried the link again from my Chrome browser logged on with my mePrism account with no success. Next I opened my drive and scanned the *Shared with me** files and found the .zip file there. Download is a function off the context menu for the file. I started the download. The file is 9.9 gig.

#### Interruption number two

##### CapTech Sprint Grooming

CapTech is finally getting around to providing estimates on their stories. John has taken an interesting approach on Jira. He has provided high level tasks that are stories. Then the actual tasks are sub-task of the stories. There was some discussion about whether to estimate at the story (task) or task (sub-task) level. We decided on estimating on sub-tasks (that is the proper approach in this case). Joseph almost steered the team toward estimating by hours, but we nipped that and are using Fibonacci story points. Grooming went on for an hour and a half and we only completed part of Sprint two.

#### Back to the data

During the grooming meeting my network dropped. I had to restart the download. When the file was downloaded, I moved it to my OneDrive mePrism folder and attempted to navigate its' contents there. Unfortunately I received a message indicating the archive is invalid. I attempted a third download on the assumption that the file was corrupted during transfer, but I still did not have luck.

![image-20200922100738486](./Images/DownloadError.png)

#### Interruption number three

Mary needs permissions to Firebase. Tim needs his permissions removed. I removed Tim from Firebase, Jira, GitHub, and AWS. I could not remove him from Slack.

#### Direct from Drive to S3

I am researching methods of transferring data directly from Google Drive to S3. So far I do not have see a pre-written method of accomplishing this. I think I will have to write my own method. This may mean writing a script that works something like ...

1. Deploy to a container or EC2
2. Script the migration
3. Create a config file, or read the Google Drive directory
4. Transfer file locally
5. Expand the archive
6. Copy to S3
7. Repeat

#### What's on Google Drive

I logged onto the Google Drive for this project. I used the following credentials ...

User name: research@datavax.io

Password: Duranduran2020

There are a number of folders out on Google Drive. 

#### Interruption number four

##### CapTech Sprint Grooming

Attempting to groom and estimate sprint number three. 

Part of this conversation was a discussion about the ML Pipeline and what it will take to "re-platform" Jinyi's projects into Lambda functions. This is a useless activity. There is no connection between the Jinyi's notebooks and the current code running in Analyze.

#### Setting up for the data transfer

I have decided to move the data using Python code as described above. I started by creating a local branch called mccann-data-transfer. In the root folder of the branch I created a folder called data_transfer. This will hold the script for the data copy. 

My first task is to write a script that can navigate the Google Drive folder structure. To do this I am using guidance found at the following link ... https://developers.google.com/drive/api/v3/quickstart/python.

The first part of this is enabling the Google Drive API. From a browser that was logged into the research@datavax.io account, I clicked the *Enable the Drive API* button. This popped up a modal dialog giving me a choice of target platform; I chose *Desktop* as this will run as a console app. Lastly I clicked *DOWNLOAD CLIENT CONFIGURATION* from the "You're All Set" panel.

| Item          | Value                                                        |
| ------------- | ------------------------------------------------------------ |
| Client ID     | 351233066343-j7jijnvdvfe31j3imb7e1frggnbsmr4v.apps.googleusercontent.com |
| Client Secret | Q6UvWNPypmAx7DGi_9eoyroA                                     |

These credentials can be managed at this link ... https://console.developers.google.com/?authuser=0&project=data-transfer-1600811675745

Next I need to install the Google Client Library with the following command ...

```bash
pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib
```

The Google quick-start page includes sample code. I created `quickstart.py`  and copied the sample code into that source file.
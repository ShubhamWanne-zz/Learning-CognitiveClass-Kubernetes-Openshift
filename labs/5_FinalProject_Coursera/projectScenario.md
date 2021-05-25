<img src="images/IDSNlogo.png" width="200" height="200"/>


# Final Project: Scenario and Review Criteria
## **Estimated time needed:** 2+ hours


## Project Overview
In this final project, you will build and deploy a simple guestbook application. You will rollout updates using Openshift image Streams. You will be rolling out a multi-tier version of the Guestbook application. You will create and bind a tone Analyzer service instance to your application and autoscale the Guestbook.

## Review Criteria
After completing the Hands-on Lab: Build and Deploy a Simple Guestbook App, you will complete the Peer-graded Assignment and be graded on the following 9 tasks.

For each of the 9 tasks you will be required to provide a screenshot and upload the JPEG (.jpg) file for your peers to review. 

**Task 1:** Deploy a simple v1 guestbook app. (**2 points**)

**Task 2:** Use an in-memory datastore for the simple guestbook app. (**1 point**)

**Task 3:** Update the simple guestbook homepage to include your name.(**1 point**)

**Task 4:** Automatically deploy the homepage update using a second image stream tag. (**1 point**)

**Task 5:** Deploy the second version of the guestbook using an OpenShift build.(**5 points**)

**Task 6:** Deploy a Redis master, a Redis slave, and an analyzer microservice.(**3 points**)

**Task 7:** Use Redis for the v2 guestbook instead of an in-memory datastore.(**1 point**)

**Task 8:** Submit entries to the guestbook and have their tone analyzed. Some simple sentences will not have a tone detected. Ensure that you submit something complex enough so that its tone is detected.(**2 points**)

**Task 9:** Create a horizontal Pod autoscaler that shows guestbook as the scale target, the current and desired replicas are three, and the last scale time is set to when the deployment was scaled up to three replicas (**4 points**)

## Next Steps
Be sure to take screenshots as per review criteria as you follow the step-by-step instructions.

## Author(s)
Lavanya


## Changelog
| Date | Version | Changed by | Change Description |
|------|--------|--------|---------|
| 2021-05-01 | 0.1 | Lavanya | Initial version created |
|   |   |   |   |
|   |   |   |   |


## <h3 align="center"> © IBM Corporation 2020. All rights reserved. <h3/>


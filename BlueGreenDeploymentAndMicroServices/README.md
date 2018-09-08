# Blue Green Deployment
We will cover this topic in two parts and than Hands-On. We will cover below two to understand and
start with Blue Green Deployments.
1. Blue Green Routing
2. Implications on App & Data Model Design

## 1. Blue Green Routing
Till we have deployed the application and scale it. Now the Day 2 problem starts i.e. consider we need to release
new feature in our application withing our SLA which is about zero downtime. Blue-green deployments are a technique for 
deploying updates with zero downtime. 

Cloudfoundry allows us to manage everything about their application: from deployment to the management of routes to an 
application. Till we deployed the articulate application, consider we added a feature in it, and now we wanna deploy
it. We will perform roughly below steps to do that it no down time
1. We will deploy our application with new version with a temporary URL
2. We will do testing over it so we can gain confidence over it
3. Then we map or bind our production URL with the instance of newer version application
4. From Step 3 some of traffic start diverting to newer version application and some of the still over the old version
5. Then we start scaling our newer application so we can get equal traffic on our application.
6. By Step 4 & 5, we get confidence that our newer version application behaving properly.
7. We start scale down our application and maintain with newer version application.
8. When we feel that we can remove our application and remain with newer one only, we can cut down our application by unbinding URL with application.

## 2. Implications on App & Data Model Design
Implication on App or Data Model design points to things need to be take care while developing new feature or fixing bug.
1. No destructive changes at field level or database level. (Never remove a field which is not in use, keep it there in class.
Same as for database never drop a column keep it,even if it is no more in use)
2. Whenever introduce a new field in database keep it nullable or provide default value if not provided.
3. If using serialization in application always provide **serialversionuid** in class, so new version and old version serialize
and deserialization can be in proper manner.
4. Here we will talk about one of the point from 12-Factor app which is admin process. Run admin/management tasks as one-off
processes e.g. migrating data process. 
5. Do make changes idempotent i.e. if we are are copy data to new field, don't delete the column or data in old column.


## Hands-On


# Application AutoScaler
We had seen manual autoscaling of application. Now we will see how we can configure the auto scale application.
PCF provides a service for this which is app-autoscaler. Let's see the description of app-autoscaler description 
with plans available.
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf m -s app-autoscaler
Getting service plan information for service app-autoscaler as naman.gupta4182@gmail.com...
OK

service plan   description
     free or paid
standard       This plan monitors and scales applications based on scaling rules every 30 second
s.   free

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```

This plan checks for the metrics and if found the min or max threshold crossed, according to that scale up and down the
application. To do this we need to follow below steps:
1. We need to create service instance of the app-autoscaler
2. Bind this to our service
3. Turn-on the service using UI
4. Configure the threshold of parameter and provide max and min instances to handle load
5. Watch events for results happening or not.

*Due to limitation of Trial account on PCF I can demonstrate over only two instance of application.

**1. We need to create service instance of the app-autoscaler**<br/>
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf create-service app-autoscaler standard ap
p-scaler
Creating service instance app-scaler in org techacademyglobal / space development as naman.gupta
4182@gmail.com...
OK

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>c
```

**2. Bind this to our service**<br/>
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf bind-service articulate app-scaler
Binding service app-scaler to app articulate in org techacademyglobal / space development as nam
an.gupta4182@gmail.com...
OK
TIP: Use 'cf restage articulate' to ensure your env variable changes take effect

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```

**3. Turn-on the service using UI and command**<br/>
**4. Configure the threshold of parameter and provide max and min instances to handle load**<br/>
![UI Autoscale enable](images/AutoscalingEnable.png?raw=true)

**5. Watch events for results happening or not**<br/>
![Scale Notification](images/ScaleNotification.PNG?raw=true)

# Application Performance Monitoring
In this section we will talk about application performance monitoring using tools like AppDynamics or new relic. For this
hands-on we need to create a account over new relic, so I am explaining theory and steps in detail, without giving any
screenshot. If we wanna monitor our application performance with third party apps like AppDynamics or new relic follow
below steps:
1. Create a user-created-service or service(if available in marketplace)
2. Bind this service with our applications
3. Restage application so it can download necessary library and build with droplet and spin up container
4. Now we can monitor these apps performance in AppDynamics or new Relic

# Metrics
PCF Metrics provides application metrics for your application directly inside cloudfoundry. It’s an innovative tool 
that combines application events, metrics (cpu, memory, network), and logs into a powerful view that can help you 
monitor and gain insight into your application’s behavior over time. Below is a high level image of Metrics view of 
cloud foundry. It is in Beta phase but still good tool for application monitoring.
![PCF Metrics](images/PCFMetrics.png?raw=true)

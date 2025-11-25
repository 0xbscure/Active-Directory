# Local Persistence in Windows Service

Windows services offer a great way to establish persistence since they can be configured to run in the background whenever the victim machine is started. If we can leverage any service to run something for us, we can regain control of the victim machine

A service is basically an executable that runs in the background. When configuring a service, you define which executable will be used and select if the service will automatically run when the machine starts or should be manually started

there are tow main ways we can abuse service to establish persistence: either **create a new service** or **modify an existing one** to execute our payload

# ASP.NET Core Background Tasks Sample (Web Host)

This sample illustrates the use of [IHostedService](https://docs.microsoft.com/dotnet/api/microsoft.extensions.hosting.ihostedservice). This sample demonstrates the features described in the [Background tasks with hosted services in ASP.NET Core](https://docs.microsoft.com/aspnet/core/fundamentals/host/hosted-services) topic.


By M.Ali El-Sayed 4/3/2019

One of the most challenged tasks for a web developer is to provide optimal user experience, long running tasks which asked the user to wait until get a response from server substantially degrade user experience.

.net core provide three techniques to overcome running long tasks in the main thread

Timed background tasks: which run the task periodically every interval provided by the developer in seconds, this replace the need for corn jobs used by PHP developers in Cpanel and schedule tasks in windows
Consuming a scoped service in a background task: which run one time at the application startup
Queued background tasks: which run in queues and can handle user action in queue so i will start with this one first and how to use it
How to use Queued background in your asp.net application 

to do this with minimal efforts we will use the sample project provided by asp.net team, you can download a copy here from my GitHub repository , unfortunately the Microsoft sample comes bundle with other documentation samples

download the project and extract it to your machine, then open the folder

Dotnet-core-BackgroundTasksSampleWebHost\Services
Then copy the following files to your services folder in your own project

BackgroundTaskQueue.cs , ConsumeScopedServiceHostedService.cs ,
 QueuedHostedService.cs , ScopedProcessingService.cs ,TimedHostedService.cs
Then open each file and change the line

namespace BackgroundTasksSample.Services 
to your own project name space

Then open your startup.cs and add the following lines

      services.AddHostedService<QueuedHostedService>();
      services.AddSingleton<IBackgroundTaskQueue, BackgroundTaskQueue>(); 
inside your services.Configure method .

The final step is to inject the IServiceScopeFactory and IBackgroundTaskQueue into your controller , so open your controller and add the following lines

    private readonly IServiceScopeFactory _serviceScopeFactory;
    public IBackgroundTaskQueue Queue { get; } 
I will assumes that your in the Home controller change your constructor to

		public HomeController(,IServiceScopeFactory serviceScopeFactory             , IBackgroundTaskQueue queue ){
            _serviceScopeFactory = serviceScopeFactory;
            Queue = queue;
        }
now every thing is ready to run long tasks in queues, use the following template to run any long task in the background and return to the user without waiting , lets assumes that you are sending an email to the user in response to an action from the user

public async Task<IActionResult> SendMail(){
		Queue.QueueBackgroundWorkItem(async token =>
		{
			using (var scope = _serviceScopeFactory.CreateScope())
			{
				// put your long task here 
				// the method will return and your task will 
				// run in the background 
			}
		});
		return View();
   }
* Timed background tasks
in your startup.cs file add the following line

services.AddHostedService<TimedHostedService>();
you are done yes really just go to the file

TimedHostedService.cs
and edit the DoWork Method like this

        private void DoWork(object state)
        {
            //your task here will run every 5 second 
        }
also you can change the interval in the start method

public Task StartAsync(CancellationToken cancellationToken)
        {
            _logger.LogInformation("Timed Background Service is starting.");


            _timer = new Timer(DoWork, null, TimeSpan.Zero, 
                TimeSpan.FromSeconds(5));


            return Task.CompletedTask;
        }
* Consuming a scoped service in a background task

This scoped service will run once at start up, to use it edit your startup.cs file and add the following lines

    services.AddHostedService<ConsumeScopedServiceHostedService>();
    services.AddScoped<IScopedProcessingService, ScopedProcessingService>();
also you are done here no more code just edit the file ScopedProcessingService.cs and change the DoWork method add your own code

for more information run the sample project as self hosted and see the console window, and see how the timed service is running every 5 second, and in the main web page of the sample project click the Addtask button several times and see how the queued tasks is running

further reading go to the Microsoft documentation here

Project source code credit goes to Luke Latham


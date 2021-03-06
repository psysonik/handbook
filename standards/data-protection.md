# Data Protection
Quality software is almost always about data. This data is important, and needs to be protected. Some best practices can help to ensure this happens.

## Minimization of Data Retrieval
In a web development world where frameworks like django do everything for us, it's very very easy to ignore how much data we retrieve unnecessarily.

Here's an example. Imagine we want to generate an HTML report where we list zipcodes for all Acme users. Out of habit we would write something like:

	users = UserProfile.objects.filter(company__name='Acme')
	return render_to_response('zipcode_report.html', {
	    'users': users
	}, context_instance=RequestContext(request))
	
There are several issues with this code block:

* It's retrieving entire user records from the DB, where we could have retrieved only zip codes
* In this process we are introducing unnecessary data to travel between the database and the application server
* It is very common in most system architecture to have database living in more secure environment than application server
* It's passing the entire user object list to the template
* One can assume that such transfer of data seems quite harmless. But in case of any exception around these code blocks may result into sensitive information written to the disk in the form of memory dump, python stack trace etc.

Instead we could change the code slightly and drastically minimize risks:

	user_zipcodes = UserProfile.objects.filter(company__name='Acme').values_list('zip code', flat=True)
	user_zipcodes = set(user_zipcodes.sort())
	return render_to_response('zipcode_report.html', {
	    'user_zipcodes': user_zipcodes
	}, context_instance=RequestContext(request))

Using the values_list filter literally reduces the SQL statement to selecting only zip code from the database. Only zip codes travel from data layer to application layer. Then the data is even more randomized by sorting and reducing to a distinct set before it's passed from application layer to presentation layer. The threat of exposure can be eliminated at the root by minimizing retrieval. Lists of zip codes by themselves are pretty harmless. But combined with users' name and other information, we quickly get into the zone of exposing PHI and PII.

## Minimizing Data Exposure
What are the different ways that we could unintentionally expose data? Unnecessary exposure of data takes place more often than we realize. We expose data in the form of:

* Application logs
* Exception stack traces
* Django settings module
* System memory dumps
* Open API endpoints resulting from using generic API frameworks
* Using database level information in URL
* Development DB practices
* Unnecessary data retrieval

### Application Logs
Application logs are one of the most common ways to expose unnecessary data. Debug level logs are great and they serve us well in identifying bugs and erratic flow of logic. At the same time it exposes a lot of information if the statements are enabled on setups with sensitive data such as staging or production server. This is why it's absolutely crucial to have appropriate log settings for each level of deployment e.g. development, staging and production.

### Exception Stack Traces
While it's not possible for us to guarantee that our code will be 100% bug free, usually exception will leave stack trace footprint somewhere in the system. Whether it's the web server or exception logger there's always a chance of seeing a dump of all local variables found at exception time somewhere in the system. Often times 3rd party packages (that come AS IS and with no warranty) might keep their own logs, and when our logic or data structure causes and exception in such packages, they might be saving our sensitive data in some logs buried in their folder structure. For example, an exception in python-rest-client might expose an internal API URL. The only way to prevent such issues is to make sure any statement that may throw an exception, is wrapped with proper exception handler.

Django stack traces are also a goldmine of sensitive settings information including DB passwords, API keys etc. Security of such information is ensured by:

* Controlling the details included in django crash logs
* Preventing such crash logs from being emailed or leaving the server
* Django Settings Module
* The settings module is probably the most vulnerable of all built-in django features. Settings module holds the information to access databases, credentials for analytics, internal API credentials, cookie names. Accidentally exposing parts of the settings module to the presentation layer could be disastrous. Disaster could come in the form of the settings dict being printed out, unintentional exposure of key value pairs when looping. For these reasons, templates should only have access to a limited subset of settings variable and not the entire module.

### System Memory Dumps
Web applications can cause servers to crash or run out of memory (has happened in our case). In situations like that, the system memory dump is usually handed off to Joe Random form cloud infrastructure support for analysis. Such memory dumps may also contain sensitive information. Preventing such cases are quite difficult but they also happen rarely.

### Open API Endpoints
API endpoints should be carefully exposed. Unless there's robust authorization in place, POST, PUT and DELETE paths should never be exposed. GET should only expose as much information as needed. The principle of least knowledge is a good rule to follow when designing APIs, specially around sensitive data. All exposed fields should be explicitly defined in the code and we should not rely upon frameworks defaults. Any kind of endpoint listing, for example Tastypie's API endpoint listing functionality should be strictly disabled. Most API access in our case should be internal and should only be allowed for logged in users.

### Database Level Information in URL
Generally considered a bad practice due to the resulting non human-readable URLs, using DB level information such as primary keys can also lead to exposure of sensitive data. Combined with an open API endpoint, primary keys can suddenly become a path the access all data in the DB. Sometimes primary keys can be used to circumvent authorization levels. URLs should not contain DB level primary identifiers. Natural keys are more appropriate in such cases.

### Development DB Practices
Having sensitive data in development DB is very common issue in web application development world. Particularly in an environment where we are all learning about the sensitivity of the data at the same time we are working with it, there increases chance of data exposure. I know for a fact that both our core DB and base profile DB dump have tons of PHI/PII in UserProfile models (including hashes of super sensitive information of real users). We need to scrub all of that. Loss of our boxes will lead to data exposure concerns.

### Unnecessary Data Retrieval
The key to minimize data exposure, is again minimal retrieval of data. Particularly when dealing with user profile information, we should be very diligent about retrieving as little information as possible to get the job done.

### Ensuring Security of Data In Process
More often than not, we are forced to retrieve or work with really sensitive data. In such cases, we can take measures to maintain the highest security available for the data being processed. An easy rule of thumb is never write such data to disk. The operating system provides various kinds of buffers that can be utlized to temporarily store and process sensitive information. If it's absolutely necessary to store the data in disk, they can be stored in encrypted form and decrypted every time before using.

Security measures such as encryption/decryption are expensive computing operations. So depending on the sensitivity of the data, it may not be possible to process them in realtime. Asynchronous worker servers are great in accommodating such cases.

### Conclusion
Exercising data protection is about conscious usage of data. It is achieved via practice of upholding protection for all kinds of data, not just the most sensitive ones. It is also dependent on the developer's wisdom and wilfulness of compliance. But being aware of our responsibilities is winning half of the battle. As long as we are always willing to consider minimization of data retrieval and exposure, it will be relatively easy to provide a reasonable standard of data protection in our system.


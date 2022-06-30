If you are reading this, you recently joined the .Net team, congratulations!!!

You are probably eager to jump on your first project, code, test, deploy, all that awesome stuff! But before you do, we would like to familiarize you with the team`s best practices as well as the agile development process.

.Net handbook is a collection of best practices the team came to through years of experience. How it looks to put theory in practice, you can check .NET example project.

Before you jump into your first project, you should try out approaches described by Handbook and implement example API applying team standards. 

Onboarding is equally important for junior colleagues that will have a chance to learn a lot of new things, as well as senior colleagues that will understand team standards. This way the team builds shared language and understanding of processes, and then strives to improve those. 

In general, the purpose of this onboarding project is to familiarize you with team processes, best practices and standards while you implement real-world apps. 

###### In this process you should:
* Understand Task, Subtask and Todos in Productive
* Do a task estimation and manipulation (In progress, Under Review, Done)
* Time tracking using Productive (track each task separately under its time log and description)
* Create a small PR, that will contain only one feature linked to a specific task.
* Ask for more questions if the acceptance criteria is not clear (some things are intentionally omitted). 



###### Onboarding API
* Tasks

Each task should define a complete feature. It should explain the purpose and acceptance criteria i.e. what is considered as a DONE.  Features should be estimated and potentially broken into TODOs. Each todo can be implemented and submitted in its own PR request. This way we keep our PRs small and easy to review. 

Hypothetical example:
 
Implement a service that will send email reminders to the Admins email address, when a new Blog is created.  Email should contain Blog ID, Blog Name and Author Name.
That way Admin will be notified and able to review new blogs as they are created. 

Above described is a Feature task. It should be broken down in Subtasks in a following way :

    * Implement email service using Sendgrid 
    * Implement queue upload service 
    * Extend Blog Create method logic to create DTO that contains Blog ID, Blog Name and Author Name and upload this dto as queue message
    * Implement queue triggered function that will read DTO and send it via implemented email service.


* PRs

Each PR should be self contained, meaning it should contain whole SUBTASK acceptance criteria. Using the above example, if you put all logic from the Feature task in PR it would be quite a lot of files to review. But if you create PR for each subtask, it is just right, as you have one unit of functionality covered by your PR . 
For example: Implement queue upload service. It will contain service logic, configuration, and unit tests. 


###### Learning materials 
This task contains valuable topics and resources each .Net developer should understand in order to be able to leverage framework features and produce efficient software.
Depending of your seniority level, you can spend more or less time on those topics, but  also feel free to extend the list with additional topics, only make sure you discuss them with your team, in order to be provided with good materials.
You can go through list at your own pace even after you finish onboarding process offically. In following months you should be able to address it and log your acitviity under .Net education service in Productive.
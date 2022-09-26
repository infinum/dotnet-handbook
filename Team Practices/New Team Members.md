If you are reading this, that means that you've recently joined the .NET team. In that case we want to say congratulations on your new role and welcome to the team! :)

You are probably eager to jump on your first project, code, test, deploy, and do all that awesome stuff! But, before you do, we would like to familiarize you with the team's best practices as well as the agile development process.

The .NET Handbook is a collection of best practices the team came to and agreed upon throughout their years of experience. Besides the handbook, you can also check the .NET example project where we put theory into practice.

### Onboarding

Before you jump into your first project, you should try out approaches described in our Handbook and implement an example API applying team standards.

Onboarding is equally important for junior colleagues who will have a chance to learn a lot of new things, as it is for senior colleagues because through that task they will learn and understand team standards. This way the team builds shared language and understanding of processes, and then works on improving those.

In general, the purpose of this onboarding project is to familiarize you with team processes, best practices and standards while you implement real-world apps.

During the onboardings process you should:

* Learn about Tasks, Subtasks and Todos in Productive.
* Do a task estimation and maintain task statuses (In progress, Under Review, Done).
* Create a small PR that will contain only one feature linked to a specific task.
* Ask for more questions if the acceptance criteria is not clear. Hint: some things might be intentionally omitted! :)
* Track your time using Productive (track each task separately under its time log and description).

#### Onboarding API

##### Tasks

Each task should define a complete feature. It should explain the purpose and acceptance criteria i.e. what is considered as a DONE. Features must be estimated and potentially broken into Todos or Subtasks. Each Todo/Subtask should be implemented and submitted through its own PR request. This way we keep our PRs small and easy to review.

__Hypothetical example:__

Let's say that you have the following Feature task:
    Implement a service that will send email reminders to the Admin's email address whenever a new Blog is created. Email should contain Blog ID, Blog Name and Author Name.
    That way Admin will be notified and should be able to review new blogs as they are created.

This task should be broken down in Subtasks in the following way:

  * Implement email service - add the service, interface, service DI registration and tests for the Email Service. Send emails using Sendgrid.
  * Implement queue upload service - add the service, interface and DI registration.
  * Extend Blog Create method logic - create a DTO that contains Blog ID, Blog Name and Author Name and upload this DTO as queue message.
  * Implement a queue triggered function - add a queue triggered function that will read the DTO and send the email using previously implemented email service.


##### PRs

Each PR should be self contained, meaning it should contain the implementation that conforms with all of the SUBTASK's acceptance criteria. Using the example above, if you put all logic from the Feature task in PR it would be quite a lot of files to review. But if you create PR for each subtask, it is just right, as you have one unit of functionality covered by your PR.
For example: Implement queue upload service. It will contain service logic, configuration, and unit tests.


### Learning materials
[Editor's note: this section should be rewritten because even I don't understand what list this is referring to.]

This task contains valuable topics and resources that each .NET developer should understand in order to be able to leverage framework features and produce efficient software.
Depending of your seniority level, you can spend more or less time on these topics, but also feel free to extend the list with additional topics - just make sure you discuss them with your team in order to be provided with good materials.
You can go through the list at your own pace even after you officially finish the onboarding process. In the following months you should be able to address it and log your acitviity under .NET education service in Productive.

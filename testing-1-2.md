# Why use and What is Continuous Integration?

Using a system like Git allows us to review code before it's
committed and deployed. Along with the benefits, which are
huge, this introduces a problem. Integration and testing coverage
become daunting to handle. That problem, like so many others,
can be solved by using good DevOps practices to take that
manual labor off of the minds of people with better things to
worry about.

Clearly, working backward to fix bugs when they've been noticed
down the line and are no longer simple to track down can be
ridiculously difficult. Easier would be to catch bugs before they
ever hit production, or even staging. Enter _Continuous Integration_ ([CI]()).

CI, executed well, is a method in which every commit triggers a an integration to a shared repository. This is then built and tested;
in this way bugs can be tracked down to their original commit and repaired smoothly.

Aside from just making things cleaner and easier to manage, CI
measurably reduces the expense and hours used to fix bugs. The risk
of introducing a breaking bug into a mission critical system
is also greatly reduced with the check, build, test, deploy method
of continuous integration.

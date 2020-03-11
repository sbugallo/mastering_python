1. The Twelve-factor app
************************

The main requirement for painless deployment is building your application in a way that
ensures that this process will be simple and as streamlined as possible. This is mostly about
removing obstacles and encouraging well-established practices. Following such common
practices is especially important in organizations where only specific people are responsible
for development (the developers team or Dev for short) and different people are
responsible for deploying and maintaining the execution environments (the operations
team or Ops for short).

All tasks related to server maintenance, monitoring, deployment, configuration, and so on
are often put into one single bag called **operations**. Even in organizations that have no
separate teams for operational tasks, it is common that only some of the developers are
authorized to do deployment tasks and maintain the remote servers. The common name for
such a position is DevOps. Also, it isn't such an unusual situation that every member of the
development team is responsible for operations, so everyone in such a team can be called
DevOps.

No matter how your organization is structured and what the responsibilities of each
developer are, everyone should know how operations work and how code is deployed to
the remote servers because, in the end, the execution environment and its configuration is a
hidden part of the product you are building.

The following common practices and conventions are important mainly for the following
reasons:

- At every company people quit and new ones are hired. By using best approaches, you are making it easier for fresh team members to jump into the project. You can never be sure that new employees are already familiar with common practices for system configuration and running applications in a reliable way, but you can at least make their fast adaptation more probable.
- In organizations where only some people are responsible for deployments, it simply reduces the friction between the operations and development teams.

A good source of such practices that encourage building easy deployable apps is a
manifesto called the **Twelve-Factor App**. It is a general language-agnostic methodology for
building software-as-a-service apps. One of its purposes is making applications easier to
deploy, but it also highlights other topics such as maintainability or making applications
easier to scale.

As its name says, the Twelve-Factor App consists of 12 rules:

- **Code base**: One code base tracked in revision control and many deploys
- **Dependencies**: Explicitly declare and isolate dependencies
- **Config**: Store configurations in the environment
- **Backing services**: Treat backing services as attached resources
- **Build, release, run**: Strictly separate build and run stages
- **Processes**: Execute the app as one or more stateless processes
- **Port binding**: Export services via port binding
- **Concurrency**: Scale out via the process model
- **Disposability**: Maximize robustness with fast startup and graceful shutdown
- **Dev/prod parity**: Keep development, staging, and production as similar as possible
- **Logs**: Treat logs as event streams
- **Admin processes**: Run administration/management tasks as one-off processes

Extending each of these rules here is a bit pointless because the official page of Twelve-
Factor App methodology (`http://12factor.net/ <http://12factor.net/>`_) contains extensive rationale for each
app factor with examples of tools for different frameworks and environments.
This chapter tries to stay consistent with the preceding manifesto, so we will discuss some
of them in detail when necessary. The techniques and examples that are presented may
sometimes slightly diverge from these 12 factors, but remember that these rules are not
carved in stone. They are great as long as they serve the purpose. In the end, what matters
is the working application (product) and not being compatible with some arbitrary
methodology.
# LIXA History

Every project has its own history and LIXA it's no exception.

## The How, The Why and the When

It was 2009 and in my professional life I had to cope with UNIX applications and 2 phase commit transactions. 
Online applications were not that bad situation because TP monitors typically provide the capability of managing distributed transactions across two or more resource managers.
But batch applications were a completely different beast:

- transforming then in "quite online" applications?
- looking for some sort of transaction manager for batch applications?
- getting rid of 2 phase commit and implementing some sort of "checkpoint/restart" logic at the application level?

No approach satisfied the different stakeholders for one reason or another. In the end I rember that company used a couple of alternatives:

- for COBOL applications, that at the end of the day behave quite similar to C applications, [IBM MQSeries / IBM WebSphere MQ / IBM MQ](https://en.wikipedia.org/wiki/IBM_MQ) was used in the role of *transaction manager*
- for Java applications, [Bitronix Transaction Manager](https://github.com/bitronix/btm) was used

I was not really interested by the second one because it was a pure Java technology for Java applications: using it with a different language, let say C or Python seemed to me a not natural choice.
About the first one, it had a lot of limitations, just to cite a couple:

- only the *bind* mode was supported and that required the application to run inside the same system that hosted the queue manager acting as transaction manager
- only one XA configuration per queue manager was possible: if you had two applications that required two different XA configurations, like for example two distinct users and passwords to access the same database, you needed two queue manager with two distinct XA configurations

To make a long story short, the limitations introduced topology and scalability constraints.

At that point I started asking myself: is it possible to create an XA *Transaction Manager* with some peculiar characteristics:

- it's **not** a *Transaction Monitor*
- it's **fully distributed**, in other words it allows any type of deployment: local or remote as preferred by the user
- it's **flexible to configure**, allowing different applications to use different XA configurations in a *natural* way

After some thinking and intensive reading of the [XA specifications published by the Open Group](https://pubs.opengroup.org/onlinepubs/009680699/toc.pdf) I established there was a positive answer to the above questions.
That exact day I started to develop LIXA, acronym of LIbre XA.

### *...to be continued...*

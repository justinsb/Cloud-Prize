Submission for Netflix OSS prize by:

justinsb, founder of FathomDB, USA

## Which Categories Best Fit Your Submission and Why?

* **Best new monkey**

I've implemented a 14 new chaos monkeys, simulating different types of failures.  I've also made it really easy to create more chaos types just by writing a script, which is run via JClouds/SSH.

Also:
* Best usability enhancement

I've fixed the problems I encountered while setting up the Simian Army, including documenting how to use the new AWS CLIs.

* Best new feature

I think that the idea of running sneakier chaos monkeys to simulate more devilish failures might even quality as a new feature.

* Best contribution to operational tools, availability and manageability	

I think that if Netflix can survive an onslaught by the 15 monkeys of chaos, then it is much more likely to remain available throughout non-simulated problems.


## Describe your Submission

> There are more things in heaven and earth, Horatio,
> Than are dreamt of in your philosophy
> 
> _Shakespeare, Hamlet_

> There are known knowns; there are things we know that we know.
> There are known unknowns; that is to say, there are things that we now know we don't know.
> But there are also unknown unknowns – there are things we do not know we don't know.
>
> _Donald Rumsfeld, Press Briefing_

The chaos monkey currently just shuts down a machine; it simulates a "clean failure".  The chaos monkey was a huge leap forward by Netflix engineering, but we've seen in a few AWS outages now that large-scale AWS failures are not clean.  This is good - things fail in new ways at AWS cloud scale; we'd be upset if the same things kept going wrong.  But we need the chaos monkey to simulate more types of failures if we want to more accurately simulate AWS failures.  We've seen network connectivity issues; we've seen EBS issues; we've seen S3 go offline.  I think the only thing we can be sure of is that the next failure will be different!

So now, when the chaos monkey selects a victim instance, instead of just shutting it down, I've changed it so that it randomly picks a mischievous monkey from a barrel of chaos monkeys.  I've added some truly evil strategies:

* _Simius Quies_: Block all networking on the instance using EC2 security groups (network failure)
* _Simius Desertus_: Null-route the 10.0.0.0 network (failure of the internal EC2 network)
* _Simius Perditus_: Cause packet loss (degradation of EC2 network)
* _Simius Tardus_: Add packet latency (degradation of EC2 network)
* _Simius Politicus_: Cause packet corruption (degradation of EC2 network)
* _Simius Nonomenius_: Block all DNS traffic (failure of DNS servers)
* _Simius Amnesius_: Null-route S3 traffic (S3 failure)
* _Simius Noneccius_: Null-route EC2 API traffic (EC2 control plane failure)
* _Simius Nodynamus_: Null-route DynamoDB traffic (DynamoDB failure)
* _Simius Amputa_: Disconnect all EBS volumes (EBS failure)
* _Simius Cogitarius_: Burn CPU (noisy-neighbor / CPU issue)
* _Simius Occupatus_: Burn disk I/O (noisy-neighbor / disk issue)
* _Simius Plenus_: Fill up all available space on the root disk (disk full error or disk failure)
* _Simius Delirius_: Kill all java & python processes on the machine repeatedly (general application fault)

These 14 new monkeys generally defeat simple Auto-Scaling-Group rules already, because the instance typically keeps running.  I know that Netflix will have more sophisticated rules, but I feel pretty confident that at least one of these chaos monkeys will defeat Netflix's current rules, and that hopefully fixing this will mean I can keep enjoying Breaking Bad and House of Cards next time AWS goes down!

Some of these chaos monkeys involve writing Java code (the ones that use the EC2 APIs), but I have also made it much easier to create a chaos monkey - it can now just be a shell script (with a trivial wrapper class).  To do that, I added JClouds integration, which has excellent SSH support built-in.  The shell script is uploaded over SSH to the victim machine, and then executed.  I think that "simplicity is a feature" - it's so easy to create new chaos monkeys that I hope many more will be created, by Netflix, myself and others.

For example, I'd like to add a strategy that null routes Eureka based services, but I'm not sure of the exact DNS names to block.  But, for Netflix, adding that is now a simple shell-script away.

Finally, to help people get going with Netflix's Simian Army, I cleaned up a few gotchas that I hit.  The wiki for getting going used the old AWS command line toolset, so I added the new shiny AWS CLI commands.  There was an issue with strict parsing of boolean configuration values, so I fixed that when I hit it.  The wiki instructions had a very complicated procedure for creating the required SimpleDB domain (it involved curl, openssl hashing for the signature, and supposedly didn't work reliably!) so I added code to just auto-create the SimpleDB bucket.  And finally, the out-of-the-box config tried to send email to foo@bar.com; if the user hasn't configured a real email I added code so that we detect that email address and ignore it.  These improvements have been merged in to the official Netflix repo (or are already on the wiki).

All code is Apache licensed.  It has all been submitted via Pull Requests to the Netflix SimianArmy project, some of which have already been accepted and merged into the official Netflix project in use at Netflix. 
The code passes the test suite, including CheckStyle tests, PMD tests and unit tests.  It therefore follows the existing Netflix code style guidelines.  Documentation has been added to the wiki, and existing installation documentation has been improved.

In terms of the categories:

* Best new monkey

For adding a barrel of chaos monkeys, integrating with JClouds and making adding a chaos monkey as easy as writing a script.

* Best usability enhancement

For fixing the installation gotchas to let others get going quickly, 

* Best new feature

For recognizing that failures are not always clean, adding support for a variety of failures, and making it easy to add more.

* Best contribution to operational tools, availability and manageability	

For enhancing the chaos monkey to simulate a whole range of new failures, meaning that a huge variety of real world failures can be simulated, so that Netflix is more likely to remain available during the next incident.

## Provide Links to Github Repo's for your Submission

My Simian Army fork:
https://github.com/justinsb/SimianArmy

Some changes already merged in to, everything is in a PR:
https://github.com/Netflix/SimianArmy

Wiki page with updated instructions (new CLI, auto-create SimpleDB domain):
https://github.com/Netflix/SimianArmy/wiki/Quick-Start-Guide

Wiki page with documentation on the new army of chaos monkeys:
https://github.com/Netflix/SimianArmy/wiki/The-Chaos-Monkey-Army


# Cloud Services, or How We Learned To Stop Worrying and Love Instances

**Our contract** with $MediumHostingProvider has been 'coming to an end' for a while now; as their customer service accrues mistakes (a missed deadline here, a miscommunication there, a forgotten upgrade) the resistance within Market Dojo to renewing the contract grows. This is in some ways a shame as they were an early partner in our journey and, had they not been subsequently bought out and overrun by a larger web host, could well have been the other half of a long and happy relationship.

There have been other concerns than the performance of our hosting company. As our platform expands, with new features each release and a growing number of clients, the demands we place on our infrastructure itself grow accordingly. Currently the Dojo operates as a single shard on a single VPS; this has been fine and workable for a long time, but our desire for resilience and scaleability is pushing us to look further afield for solutions - there's only so much peace of mind a good backup system can produce. 

It was suggested, and indeed attempted, that we could improve performance at peak times by increasing our various provisions. The expectation was that we would see increased responsiveness with an increase in memory in particular, as although the number of CPU cores available makes some difference the Rails framework is less parallel than one might like. This solution, however, struck the team as decidedly suboptimal. In particular, the difference in usage between times of high demand (such as the Monday morning rush of auctions from our larger clients getting the head start on the week), and times of low demand (such as that experienced late on a Sunday afternoon), grows linearly with the number of users since the latter is negligibly close to zero usage by comparison. This would mean that simply increasing our VPS size was more apt to waste money and electricity than to provide a noticible all-around benefit to both us and our clients.

Better solutions are out there. There are some very large cloud providers offering both IaaS and PaaS for various needs, most of which are intended to be scaleable. After some initial winnowing, we settled on a shortlist of three: IBM Bluemix, AWS Elastic Beanstalk and Google App Engine for PaaS, and the same companies' offerings for IaaS - SoftLayer, EC2 and Compute Cloud respectively. 

We had a number of requirements to consider in order to bring this down to a final choice.

 - First, and most obvious, was *price*. 
The ideal solution would not cost substantially more than our existing platform. We set a cap of about 15% increase; it's always worth being realistic about the fact that if one expects to gain something, there's often a price to be paid.
 - Second, *equivalent offerings*. 
With our existing capacity in hand, we wanted to be sure that we could have similar provision without having to customise too much; as it turned out this wasn't an issue, as all the providers offer similar instance shapes and sizes.
 - Third, *user-friendliness*. 
Time spent on ops is time not spent on developing features, fixing bugs or supporting clients. It's never a good idea to begrudge maintenance time, but for a platform to be appealing we had to be satisfied that it would not substantially increase the proportion of our day to day work devoted to keeping the metaphorical machinery oiled. A personal preference for a platform with a good and well tested CLI available was also on the checklist; the more that could be done without opening a browser tab and hunting down a phone to respond to the inevitable multi-factor authentication request the better.
 - Fourth, and uniquely to the PaaS offerings, we needed them to have either *good buildpacks* or a *flexible deployment system*, depending on how the platform was configured. 
Moving an app from self-hosted to such an environment involves dealing with the assumptions of that environment. That's all fine, good and expected, but if 'dealing with the assumptions' means rewriting large chunks of either the app or the configuration to fit that particular service then we would have to move on. Time, after all, is always in short supply. Essentially, this was the concern of *backward compatibility*. Portions of the Dojo are in the process of being refactored, improved and reconstructed, others have already been seen to, but others still are yet to be addressed in this cycle. It's an undeniable fact that any piece of software older than one or two years will be at least partly outdated, unless it is particularly small or has a particularly deific development team behind it. For our purposes this ruled out any platform which enforced restrictions on versions of Ruby, Rails, or any of the various Gems we rely upon.

The decisions became easier from there on. There were tables:

||IaaS|PaaS|
|---|---|---|
|Benefits|Full control|Someone else administers|
||Choice of web server|Provided managed web server|
||Billed by spec (predictable)|Billed by usage (auto-flexible)|
||Powerful CLI (Usually)|Easy GUI (Usually)|
||Typically cheaper than PaaS|Low technical knowledge barrier to administration|
||Can be expanded in real time to cope with changes in demand|Can expand themselves in real time to cope with changes in demand|
||Easy to automate setup via images or Puppet|Easy to automate setup via config file and push hooks|
|Drawbacks|Have to administer manually|No control over software|
||Responsible for software and dependencies|Hard to install additional dependencies|
||Requires technically competent administrator to make any changes|Narrow access routes|
|||Typically more expensive than VPS|
|||Can require official technical support to get anything major done|

Tables make everything better. There wasn't a justifiable reason for a good chart or map in this particular investigation, but there were plenty of data to work with all the same. 

There were also a great many emails. An important thing to consider when you're contemplating a move of this kind, especially where it affects the nature of your underlying infrastructure, is how your larger or more paranoid clients might be disposed towards the changes. 

Such factors as location of data come into play; in our case some of our clients have a strong preference for their data being stored in the EU, subject to the Data Protection Act. This rules out the major US datacenters. Fortunately, all the services under consideration had datacentres in either the UK, Ireland or, for reasons best known only to them, Belgium. We also had to consider encryption and colocation of data, such as would occur in a shared database or shared hosting. An encrypted-at-rest VPS is the very minimum expected, however, so those were not significant barriers.

Had we been looking at smaller hosting providers, or at self-hosting, we might also have had to consider redundancy of hardware in addition to the redundancy of data; one of the major advantages of using the large cloud providers is that they have all those concerns in hand; barring natural disasters, enemy action or Outside Context Problems they are unlikely to suffer full loss of data; mitigation of at least the first and in many cases the second can be performed relatively simply by having backups in a datacenter in another country.
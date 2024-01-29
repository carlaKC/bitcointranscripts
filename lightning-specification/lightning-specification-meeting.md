---
title: "Lightning Specification Meeting"
transcript_by: carlaKC via tstbtc v1.0.0 --needs-review
tags: ['lightning']
speakers: []
categories: []
date: 2024-01-15
---
Speaker 0: 00:00:00

I'm

Speaker 1: 00:00:03

going to repost the topics.
First of all, it looks like we still have a mailing list.
I don't know how much we can rely on that.
But in the meanwhile, nobody has sent any email on the mailing list.
But I guess we should be migrating to Delvin Bitcoin for now.
Has someone experimented with running a discourse instance somewhere else?
I think it was Laolu who was supposed to do that.
Yeah, so I guess nobody.
So let's switch to the v3 transaction topic.
Maybe Greg or someone else, if you want to give us an intro and tell us what you're expecting and what kind of hack you'd like to reach and what exactly we should be working on to make sure that we are able to reach an hack and say exactly what we need and what would be helpful to us.

Speaker 2: 00:01:00

And Cori, do you wanna pick that up?
Start?

Speaker 3: 00:01:04

Yeah, sure.
Hello.
Can you hear me?

Speaker 4: 00:01:07

Hey, perfect.

Speaker 3: 00:01:09

Great.
So I'm going to share a doc that Greg and I put together for this.
Yeah, I'll go first and talk about the roadmap for what we're trying to achieve here.
There's some really nice stuff that we get short term and then some really nice stuff that we get long term that kind of addresses a lot of issues that I'm sure y'all are aware of.
So the short term stuff is v3, one parent, one child package relay and ephemeral anchors.
So the idea there is that you could switch to like zero fee commitment transactions with one anchor that either party can bump and your commitment transaction plus fee bumping child can replace each other in mempool.
And then we can get rid of cpfp-carvout.
They should propagate with one parent, one child package relay.
Hopefully it's much more efficient with ephemeral anchors.
And yeah, and then long-term, there is a very nice cluster mempool proposal, which I'm sure Suhas can talk more about.
And that kind of addresses a lot of the more fundamental mempool problems that just make things really difficult.
And then we can have more general package relay built on top of that, like for general ancestor sets and Use cases wise, hopefully after that you could have things like batch debumping and whatnot.
So what we wanted to do is make v3 and formal anchors as useful as possible for the Lightning use case.
We wanted to talk about CPFP carve out going away.
And yeah, is that sound about right, Greg?

Speaker 5: 00:03:11

Sounds right.
So the CPFP carve out, How many people know what that even is?
Is there like, oh, there's a few people.
So yeah, I've got Ben, Matt, a few other people for sure.
But basically it's this, yeah, yeah, it's on holiday, yep.
It's a kind of descendant limit and ancestral limit based carve out for child, place, or parent where you guys use two anchors today and it lets you spend your anchor without having to RBF their package essentially.
But in a cluster memploid world, that might be a little difficult.
Suhas, do you want to give that spiel since you're more most practiced?
Go on.

Speaker 6: 00:04:00

Did he drop?
Sorry, I had to figure out how to unmute in this software.
I mean, I don't know how much context people have on mempool design and work that's going on.
But I guess the very short summary is that in order to solve a lot of the problems we have today with the mempool, we would like to be able to keep, currently our mempool code does not keep a totally sorted mempool.
We don't have a total ordering on transactions in the mempool.
And that makes it hard for us to evaluate kind of the mining score of a transaction, which in turn makes it hard for us to design protocols that would like to know what the relative mining scores of transactions are.
So we think we can solve this by putting a bound on the size of connected components of the transaction graph within the mempool.
By doing that, we think we can keep the mempool kind of fully sorted by sorting each connected component separately.
And this kind of allows us to have really nice algorithms for block creation, for managing mempool eviction, and for managing RBF, and ultimately for things like package validation and package review, we think.
That's further afield.
The thing is that having a bound on the size of a connected component of the mempool is a different concept than having a limit on the number of descendants that a single transaction can have.
And so I don't believe it's possible to come up with a similar carve-out kind of idea like we have for managing the descendant limit today.
My quick intuition on this is imagine you have, you know, say 90 transactions that all have their, you know, two anchor outputs that you'd like, you know, either party to be able to spend.
But one person is one of the parties on all of those and constructs a single transaction that spends all of their outputs.
Now you've got a cluster of size, say, 91.
And in order to allow every other output of every other transaction to be spent, your cluster could get as big as, say, 180 or 181.
So I'd like to keep a cluster limit that is sort of bounded.
And doing that in a way that is both achievable and doesn't have a lot of waste where you artificially restrict limits to be very low to account for some weird carve out behavior that could may blow things off seems tricky.

Speaker 4: 00:06:47

I think it may be worth, before we get too far into this, I think it may be worth pointing out that I believe the only software that actually uses the card right today is Eclair.
The rest of Lightning does not.

Speaker 5: 00:06:59

Is that, so Core Lightning, for example, does blind.

Speaker 2: 00:07:03

Everyone uses it.

Speaker 6: 00:07:04

Yeah, if you do a

Speaker 5: 00:07:04

blind spend of the counterparty's latest commitment transaction off the anchor, that could use it.

Speaker 4: 00:07:12

Right, I didn't think anyone but Eclair did that.

Speaker 5: 00:07:14

Core Lightning does it.
In my research for this topic, when I implemented the alternative spec, I said, oh, it actually does that.

Speaker 2: 00:07:21

Well, I'm surprised to hear that.
I mean, you don't need to explicitly use it.
You sort of get it as like a side effect of the rule existing.
Are you talking about targeting some specific spend or?

Speaker 4: 00:07:28

The only reason that Lightning cares about it is if you assume your counter party has broadcasted a state and you try to RBF that state.

Speaker 5: 00:07:39

The latest state.

Speaker 4: 00:07:40

Not necessarily the latest, you could assume.
You mean CPFP rather

Speaker 2: 00:07:44

than RBF.

Speaker 5: 00:07:45

Okay, I don't think anyone does older versions.

Speaker 2: 00:07:47

Well, we do a thing that we're changing now, where at any point, we're not necessarily sure which one they broadcast.
Like we'll actually try to potentially CPFP three different versions, right?
Which is our version, their version, and their sort of un-revoked version, which may exist.
That's what we

Speaker 5: 00:08:01

do.
Yeah.
So YouTube

Speaker 4: 00:08:02

as well now.
You didn't use to, correct?

Speaker 2: 00:08:06

We've done that the whole time.
We're sort of like reexamining this behavior now as far as like, you know, doing it less.
I mean, you know, the current state of things is a bit fickle, but you know, we had some complaints basically about we were being a little too aggressive when we were shooting these anchors.
So now we have a different thing now where we only try to sweep one, we only try to sweep one when there's actually a fee deadline at hand.
Another thing that we're doing now is like, maybe this related, it's like, sort of like this interaction with like fee filter, which I think maybe Matt pointed out maybe a month ago show, where it seems like because the fee filters existence, we can only rely on just our own local mempool as far as inclusion basically when we're in broadcast.
Now we're trying to look at our peers fee filter as well because we realize that's the relay path, the relay path is what really matters.
Because otherwise, our mempool maybe is too constrained because we have the default values.
We're not actually evicting the fee level that they were evicting basically.
This seems to be a wider spread issue that we're trying to investigate a little bit more as far as a propagation and things like that.
Because otherwise you think something can propagate, but it's really not going past any of your peers, maybe not some of them.
This is a behavior as well where even though we have the default policy, our mempool is like a 200 megabytes, something like that itself, and that causes us to have a much lower eviction rate.
We think we can get a 10-sided byte into mempool, looking at our min mempool from Bitcoin.
It's really much harder now, but that's an aside.
I think that's something we're trying to fix right now because it's causing a lot of issues because many of the force codes we've been seeing over the past month or so are just due to cascades basically.
And so in LND, we have a parameter called mempool max anchor, which basically sets the max anchor rate that we'll set the anchor to basically, and that was like 10th out of byte because, hey, we're in the zero fee interest rate environment before this.
We're trying to make that automatic now.
We're trying to make that automatic base off of the fee filter of all peers and also the mempool that we see locally as well.
Because otherwise, it's something that people need to set, and they're not setting it.
And we didn't know what to set it to, but I think now we feel like we have a valid that we can use to automatically set it.
And that should just help out a lot of just issues going on right now because people aren't studying this value and they can't get something into the mempool.

Speaker 5: 00:10:04

All right.
Can I interject real quick?
Yes.
That's all reasonable.
I guess the point was, people are actually using it.
The only people that seem to be using it is Lightning Network folks, and we want to make sure that we're not rug pulling you guys on taking out something useful without giving something useful in kind of return, right?
So that's kind of- But

Speaker 2: 00:10:24

how can it go away, right?
Because it's like policy and policy is like a gradient,

Speaker 5: 00:10:28

you know,

Speaker 2: 00:10:29

like there'll be 27 or something like that, but like we'll still be relying on it.

Speaker 5: 00:10:32

So do you mean for the up?
Okay, let's say let's say Let's say become core 27 get rid of it.
It's not but let's just say for instance, right?
It's all the update lag would be there right like The old nodes still around so things wouldn't still propagate using the old carve out, but over time as the network updates, then that would be going away, right?
Specifically speaking, if people start upgrading to cluster mempool based implementation sometime in the future, over time the carve out would basically dissipate in effectiveness, right?
So, that said...

Speaker 4: 00:11:07

Specifically package relay, right?
Package relay completely obviates the need for it without breaking any existing code.
As long as package relay kind of is seamless and magically exists and we don't have to change anything.

Speaker 2: 00:11:21

Yeah, the relay path exists.

Speaker 5: 00:11:23

On this document, I have some like possible futures we could do and we, I mean, you guys, in the Lnspect universe that I think are reasonable.
Some are just using package RBF, some are using shared anchors that are keyed.
You can do that too.
So it's like, you know, min size output that both of you could spend, or you could just have, or a single keyless anchor using formal anchors.
So there's some choices there, and we just want to make sure that with zero fee commitment transactions and all those nice things you get with it, that you guys would be incentivized to update to that and then wouldn't be relying on CppCarveout anymore.

Speaker 4: 00:11:59

That's essentially the story.
Why does there, there doesn't need to be a change to the Lightning commitment transaction format at all to remove zero fee anchors as long as we get package reliant.

Speaker 5: 00:12:12

Package RBF, you mean like a

Speaker 4: 00:12:14

package RBF, correct?

Speaker 5: 00:12:15

Yeah.
So that's another wrinkle here, right?
Let's see.
For package RBF, prior to cluster mempool, we're pretty limited.
So the design we're going through right now is basically you have one parent, one child conflicting with any number or up to 100 or up to 100 cold transactions that are all within cluster size of two.
So in this case, where if you're limited again, conflicting against a topology of size two, then we can compute like much better the incentive compatibility of the RBF and accept it or not if it makes them pull better.
It's actually, yeah.
So there is concern with short-term with being able to restrict the topology of our comparisons, I guess is my point.
So you guys don't even need...

Speaker 4: 00:13:08

Are you saying that...

Speaker 6: 00:13:08

If you had...

Speaker 5: 00:13:09

Sorry, to back up a little bit, you'd be happy with just package RBF, nothing else.
So you'd be like single anchor, like single anchor key.

Speaker 4: 00:13:19

Yeah, we would just ignore the other anchor basically, or code that doesn't ignore the other anchor would be come dead largely.

Speaker 5: 00:13:27

So Suha says, I don't think package RBF is enough to replace CPP carve out.
I think it was in the context of when you do package RBF, you have to, you have to, oh yeah, so, okay.
There's the other issue where for package RBF, the child, the children of your counterparty's anchor can be too large for pinning perspective?
125 rule three, correct?

Speaker 6: 00:13:55

There's two things that-

Speaker 4: 00:13:56

My comment hand waves a lot of that away.

Speaker 6: 00:14:01

I think there's two different issues there.
There's a pinning issue with, I guess if I understand you, what you guys are getting at, Matt, tell Perry if I'm wrong, you're suggesting that with package RBF, you would spend your anchor output with a high enough fee that it can replace whatever it needs to.
Is that right?

Speaker 4: 00:14:19

Correct.

Speaker 6: 00:14:19

But so we don't currently have a good algorithm for sibling eviction, but I mean maybe we come up with one in this particular case, but then you're still back to all the pending problems that I thought plagued this kind of solution.

Speaker 4: 00:14:37

So, okay, there's a few problems here.
The, the hack, the, even in a world where you're assuming that your counterparty, oh never mind, yeah I mean I'm basically assuming that, I'm basically hand waving away the pinning issue, assuming there is some alternative better outcome right so maybe something like top of mempool allowing eviction and in some of these cases or something like that, where like, like at some point we're offering some very substantial amount of money for miners if they're willing to evict that other package.
And the fact that we're offering something that is substantial more than it means like something needs to change and somehow the policy needs to let us do that basically is what I'm saying.

Speaker 6: 00:15:38

I don't think we have a proposal that is anywhere near the finish line on achieving that kind of a goal.

Speaker 4: 00:15:48

Right.
And I don't think it's realistic for...
So, I mean, then you're talking about like a different commitment transaction format.
So, first of all, Lightning doesn't currently have a protocol and people are working on one to change commitment TX format for an existing channel, right?
So like we're years away from being able to deploy a new commitment TX format and saying like, yep, okay, we're like, anyone who cares about it has upgraded, we can just stop the old one.

Speaker 6: 00:16:20

Can I clarify one point on that?
You specifically talking about going from two anchor outputs to say one ephemeral anchor output.
Is that what you're referring to?

Speaker 4: 00:16:27

Sure.
Or yeah, or whatever, they're in zero fee, whatever.

Speaker 6: 00:16:32

Well, cause I think there's another option which could be to simply make a, like the V3 proposal is to allow for transactions to opt into a topology restriction, which is that, you know, you can have at most one child.
Now we can tweak what that topology restriction is.
For example, if you wanted to have a commitment transaction with two anchor outputs and that was it, and each anchor output could be a single spend with no other in-MEMPL parent, so you have a cluster of at most size three.
I think that would be consistent with the cluster-MEMPL design and still allow us to get rid of CPFP carve-out, well, without introducing any pinning issues.
Unless Greg, correct me if I got that wrong.

Speaker 5: 00:17:14

But Can you say the last part real quick?

Speaker 2: 00:17:16

I think

Speaker 6: 00:17:16

I didn't hear it.
If you had some variant of the v3 proposal, instead of opting into a one parent, one child regime, you opted into a one parent, two children regime.
That would still kind of resolve, It would eliminate the need for CPFB carve out, I think.

Speaker 4: 00:17:33

Yeah.
So yeah.
So to be clear, my comment about the time required to get a new commitment transaction format includes like, well, all we have to do is change the version to 4.
That's still.
What would

Speaker 2: 00:17:46

the format change be?
Is that just getting rid of anchors or is it something entirely more in-depth?

Speaker 5: 00:17:51

I have a Strawman spec on number 6.
You can see the Delta.
It's mostly get rid of update fee, change n version number to 3.
That's Mostly it.

Speaker 4: 00:18:02

You can just take a look.

Speaker 2: 00:18:03

Why do you, oh, you get a fee because construction is always zero fee now?

Speaker 5: 00:18:07

Yes.
So you're left with, yeah, there's no fee negotiation during channel operation.
You're relying completely on the anchors spends, right?
And this implementation uses not ephemeral anchors, but in principle, it doesn't super matter.
You can reintroduce a minimal size dust anchor that's spent by one party pretty easily.
That's not the complex part.
And it took me about two days to get some tests working on Core Lightning, implement it myself.
There's the one caveat being I didn't get the RPC calls handled correctly, but it was creating the right kind of transactions, generating them, trying to publish them.
So it's more of a restriction in format size or format than expansion.
It's not any more complex per se.

Speaker 4: 00:18:59

Yeah, I mean, in principle, these changes are doable.

Speaker 5: 00:19:05

Yeah.
But it seems to

Speaker 2: 00:19:06

me like the bottleneck would be Bitcoin D27 or whatever that is.

Speaker 5: 00:19:10

Oh, yeah.
No, no.
So the cutoff here, right, is we're talking about this timeline.
So it's a cluster mempool to get that out is kind of blocked and making sure that people are okay with CPV carve out going away.
That's, that's the timeline there.
So we don't want to rug pull people on that timeline.
That's still a ways out.
Like, You know, it's a ways you can ask Suha specifically, but I think it's quite a ways out.

Speaker 4: 00:19:36

Right.
And I think, I mean, if we're talking about changing the format to assume.
So basically what you're proposing is we were allowed a new commitment transaction format.
We start using it today.
So a Bitcoin core rolls out some kind of limited package RBF.
Then we start using some new, you know, then we wait six months for the network to upgrade.
Then we start rolling out some new lightning commitment to X format.
We wait a year or a year and a half for people to start, to have actually upgraded and the channels migrated.
And then we rug pull the old format basically.

Speaker 5: 00:20:20

So as you can speak to timelines there, but yes, there is some sort of staggered timeline here where people have to have time to upgrade rollout actually update their software they're running software, all those things.
Software, all those things.
If you just wanted minimal, if you just said like, oh, I don't care too much, we just need limited package RBF that's sometimes pinnable, right?
If that's like the minimum bar you want, then we can do that faster.
There is the one caveat that again, I was talking about like giving pre-cluster mempool, giving a package RBF that isn't like bananas when it comes to incentives.
And so that's where that restriction, I can link it offline, but there's current work.
There's work for cluster size to package RBI.

Speaker 4: 00:21:07

So can we do something completely batch it and like have policy match lightning commitment TX formats?
Like, oh, if the commitment, if this transaction has two outputs that happen to be, whatever it is, 400 SATs, then it opts into TX version 4 and it gets all of these specific things so we get package RBI.

Speaker 5: 00:21:33

I mean, you mean, you mean

Speaker 6: 00:21:36

we basically opt into new policy rules implicitly rather than like.

Speaker 2: 00:21:41

Yeah, you're saying pattern match on the current transaction format.
And give us

Speaker 4: 00:21:44

a good look at the Lightning commitment transaction format.

Speaker 6: 00:21:48

Look, I'll take anything that gets through the carve out.

Speaker 4: 00:21:51

I mean, we could do that today.
That's not, that wouldn't require any lightning changes.
Bitcoin Core can move at its own pace.

Speaker 6: 00:21:58

Well, that breaks things for you guys though, right?
Because if, I mean one of the things we would do is we'd restrict the spends of such a transaction so you can no longer spend it out arbitrarily.

Speaker 4: 00:22:07

So

Speaker 5: 00:22:10

if Matt is saying he doesn't care about pinning then all we need to do is give something that it gives package RBF.
I think that's his point.

Speaker 4: 00:22:18

No, I don't know.
Not quite.
Because part of the answer to pinning today is using the card out.
Yes.
It doesn't solve all pinning vectors, but it solves some of them.
So if, if, I mean, I don't know how everyone's wallet works, but you know, you could imagine a world where we say like, all right, we restrict the types of anchor claims.
I mean like, so, so to be clear, in order to solve...
So, for a Lightning node today that is using the carve-out to work, the carve-out CPFP transactions must be only confirmed inputs.
If we were to match for TX version 4, that is based on just looking at the type of transaction and assuming it's Lightning, then we would similarly restrict ourselves to also on your own anchor spends only confirmed inputs.
But Lightning nodes should already have logic for only confirmed inputs because they need it if they're going to try to claim a counterparty anchor.
I don't know if that would impact our users, it probably would, but we could look into that.
So I don't know what other people think about Like what if we just restricted anchor spans to only confirmed inputs?

Speaker 2: 00:23:50

Yeah, I'm pretty sure we do only confirmed inputs.
Certain mobile wallets like to do a bunch of zero-com stuff, and we discourage them from doing it.
But at least like default, if you use like bump fee or anything like that, you know, it's our automatic sweeping stuff, we always do only confirmed inputs.

Speaker 5: 00:24:03

So what about the case where you want to use the thing like the carve out?
So you're kind of hardy as like made a large child and then you're unable to package RBF because you don't like the, like the minor has that a competing transaction, but you don't.
So you're trying to package rbf but you can't actually transmit it that's kind of where

Speaker 4: 00:24:26

we don't care as long as it's like as long as it gets confirmed And if it's not sufficient value to get confirmed, hopefully the rules that restrict it allow us to bump it, bump R's into getting confirmed.

Speaker 5: 00:24:42

No, I mean, I was making the point that there's plenty of situations where if you don't see your common parties commitment transaction, you only see your own and they're pinning you using their, their anchor outputs, but it's kind of hard to put a whiteboard here.
So you do.
So My

Speaker 4: 00:25:00

understanding of this proposal was basically we would restrict anything spending a lightning, something that looks like a lightning commitment transaction to not make it pinnable, quote unquote, where not pinnable means like, you know, one transaction that can spend it up to a certain size with only confirmed inputs.

Speaker 2: 00:25:22

Okay, so

Speaker 5: 00:25:23

you're saying like v3-like but imputing it onto template?

Speaker 4: 00:25:28

Yes.

Speaker 5: 00:25:30

And then doing sibling eviction-like thing in this narrow band, basically you're opting into this.
So the counterparty could make a child of size blah but this blah is limited and your spend of your anchor can implicitly RBF that.
Is this what you're talking about?

Speaker 4: 00:25:54

Yeah, basically.

Speaker 5: 00:26:07

Yeah.
Peanut Gallery is going wild here.

Speaker 2: 00:26:10

Yeah, chat popping off.
I mean, it depends.
Like I was saying earlier, we do have quite a lot of stuff in the pipeline generally, protocol wise.
I mean, that's like another thing just to be able to like, hey, what are you working on today?
What are their goals for the year?
But it depends.
If everyone makes it the number one priority, I think we can get it done.
But there's just a lot of other stuff lingering.
But to hijack for a second.
Oh, go ahead.

Speaker 7: 00:26:32

It can't

Speaker 4: 00:26:32

start until Bitcoin Core has shipped something and then also the network has upgraded.
And then also lighting.

Speaker 5: 00:26:40

Part of the pause.

Speaker 2: 00:26:41

Yeah, I think Matt's right in that, like us doing the format won't be the naturally the bottleneck in the deployment pipeline for that sort of something like that.

Speaker 3: 00:26:48

So what if we had v3 in like 27 or something?

Speaker 2: 00:26:52

But doesn't that require like all of your effective relay peers also upgrade as well?
You know, in order to effectively propagate?

Speaker 4: 00:26:59

Yeah and from you to the miner.

Speaker 3: 00:27:02

Yeah, and that takes time.

Speaker 5: 00:27:04

So from our perspective, it's like, if we think that we're rug pulling you guys, then we won't deploy something until we know.
And there's also a lot of other people who have their own varied opinions that have weighed in.
So we need to know kind of like, are the people who are actually gonna use it, use it?
And what is that?
So that's kind of where we're at.

Speaker 6: 00:27:25

I mean, I think also there's like a little bit of a chicken and egg here.
I think we wanna make sure that if we're gonna deploy, deploy something like v3, so that Lightning has an alternate solution to what Carbout provides today, we should make sure we design it so that Lightning community will actually use it.

Speaker 2: 00:27:44

And So

Speaker 6: 00:27:45

I think that's why we just need to know whether this is worth doing.

Speaker 1: 00:27:51

For what it's worth, the changes on the Lightning side are really simple.
I've had a branch for a long while that just changes the commitment transaction to zero fee and not a single income, but that would be easy as well.
It's really minimal, and then it lets you clean up a lot of stuff in your implementation.
It's only the upgrade path that may require some more work, but making the new commitment transaction format is really trivial.
So I'm all in favor for doing that.

Speaker 5: 00:28:18

Okay, so Matt says, can you say that out loud?
V-L-O-L, I'm not sure that means.

Speaker 4: 00:28:23

Yeah, I mean, I was only, so the only reason I pointed this out is basically like, it would allow both of, It's not that we wouldn't then switch to using v3 and like Bitcoin Core could eventually remove this stupid matching garbage.
It's that we could both move forward in parallel and Bitcoin Core doesn't need to wait for lightning to do anything or lightning node operators to do anything.

Speaker 5: 00:28:55

So you'd still have this, let's call it, you know, proto update, you'd still have update fee, all the protocol would be the same, we'd still have issues with mempool min fee, but you'd get around the need for a CPFE carveout, this is the idea.

Speaker 4: 00:29:09

Basically, we remove our...
Bitcoin Core makes a small change, but does like...
So you do v3, But then also you apply the v3 rules to this other template matching.
And then separately, Lightning starts fixing all of this stuff because now, hey, we have v3, we can fix this stuff, but it doesn't block cluster mempool.
Cluster mempool can move forward at a pace where as long as Lightning nodes have a path from themselves to miners using v3 or v3 template mashed, then it doesn't matter, Bitcoin Core can ship cluster mempool.
So, in the meantime, of course, Lightning wouldn't be able to be free.

Speaker 2: 00:29:57

I like the idea of Bitcoin Core development not being blocked on

Speaker 6: 00:30:01

kind of lightning, you know, spec changes.
Do you think that the template matching idea you're proposing is viable in the sense that you can come up with something that would just match what Lightning's doing today and it's not, it has extraordinarily low probability of matching anything else?

Speaker 4: 00:30:19

I think so, because you have these like two outputs that are both exactly, what is it, 400 and something SATs?

Speaker 2: 00:30:28

Yeah, it's like 483 or something like that, Yeah.

Speaker 4: 00:30:30

That by itself almost feels like it should be fine.

Speaker 2: 00:30:36

Also, Quidditch also set both the sequence and the lock time at all times.
Like there's like other smaller things that look weird.

Speaker 5: 00:30:43

Yeah, ideally it'd be a very easy check, But I think you could.
There's the theoretical possibility that somebody's doing something that looks like a unilateral close on Lightning but is not.
I've never heard of that, but that'd be the one theoretical risk.

Speaker 4: 00:30:57

Well, and specifically, they would still propagate.
They just wouldn't be able to build a large chain of transaction spending for that.

Speaker 5: 00:31:05

So would you also, so, okay.
Yeah, I'll, I'll let this conversation finish.
Cause number four is like a follow-on to regardless of how we want to deploy this, what would it look like?
Question.

Speaker 4: 00:31:23

I mean, I think to be clear, we need sign off from everyone that their anchor spends, do not spend unconfirmed inputs.
That's something that we need basically sign off all of the lightning implementations today.

Speaker 2: 00:31:39

Now

Speaker 4: 00:31:39

it may be the case that it's already there because they need that logic to use CPFP carve out.
So using CPFP carve out, they already need this, But we just need to make sure everyone's on the same page there before we do something like this.

Speaker 2: 00:31:54

Yeah, so with the case of LND, I have to double check, but we shouldn't be using encoded ones for flumping.
But maybe there's something like where do I think of someone doing like a zero conf, funding off or something that we're sweeping, which happens today unfortunately.

Speaker 5: 00:32:07

Yeah, the real problem comes if you try to do a child pays for parent of chain with like multiple times, right?
So a chain of size three, say, or two attempts at child pays for parent.
It's fine if first child per parent is high enough to succeed because it'll just confirm and you just keep going.

Speaker 4: 00:32:26

Yeah, basically, we push people into RBF of the CPFP.
I think that's OK.

Speaker 5: 00:32:31

OK.
Any other high-level questions, Suhas?

Speaker 6: 00:32:37

I guess it sounds to me like there's a pre-consensus forming around V3 and its rules being a good idea and useful for lightning in the long run, but not necessarily a firm anchors.
Is that what I'm hearing or is that not, is that too strong a statement?

Speaker 5: 00:32:57

It's changing at all for now.

Speaker 2: 00:33:00

Yeah, I'll profess to not be caught up on firmware anchors enough to actually have an educated opinion.

Speaker 5: 00:33:06

Yeah, just check out the document, drop a comment if you like, it's there, everyone should be able to comment.
It's just a way of talking about the future solutions.
And one other bike shedding thing is that the way we're limiting pinning, this is number four on that doc, the way we're limiting pinning is by picking an arbitrary number that's much, much less than the normal package limits and virtual bytes size and saying, if you want to see PFB, it must be only this big.
And this big right here is one kilovirtual byte.
It's an arbitrary number.
And so this is like a potential bike-shitting vector that, for good or bad, right?
Basically, if you guys wanted to opt into this regime, even implicitly saying, right, how big do you try to, how big do your CPFPs look like, right?
How many virtual bytes do you expect those to be, right?
Cause this is kind of, if you need 100 kilovirtuobytes, then we're back to where we started, right?
But if you only need one kilovirtuobyte tops to make an effective, you know, RBF of your CPFP, then that's kind of like the limit.
And there's other limits too you could pick.
Does that make sense?
That's number four on the stock.

Speaker 3: 00:34:19

Yeah, so the question is kind of the smaller we make it, the more pinning protection you get.
So I guess but I mean, the smaller we make it, the fewer UTXOs you can use for your fee bumps.

Speaker 2: 00:34:30

Right.

Speaker 3: 00:34:30

So I guess the question is, what's the kind of, sorry, what's the smallest we can make it where you'd still feel comfortable that your wallet's going to always be able to fund your fee bumps?

Speaker 5: 00:34:42

And in the future, we can issue.

Speaker 4: 00:34:45

I was just going to say that the other issue is anchor, each TLC outputs, right?
Because you can spend like, you know, up to 400 HTLC outputs that were in the commitment transaction in one follow on transaction.
And that has to be something that can't be pinned.
It's not the same because it's not like an anchor.
It's not this like two transactions, but it also has issues here.
I don't know how that fits into v3.

Speaker 5: 00:35:13

I mean all the, sorry, You have the, let's talk about parent and child transactions.
There's the commitment transaction, which would be the parent.
And then there's the child who pays, who spends at least the anchor output, right?
I think you can spend the counterparty's commitment transaction with your two remote output.

Speaker 4: 00:35:31

No, this is not even anchors, right?
Sure.

Speaker 5: 00:35:34

Well, I'm just...
Oh, you're talking about HTLC pre-signed transactions?

Speaker 2: 00:35:38

Yeah, he's talking about like, screeping HTLCs either due to a breach or just due to like, you know, success or timeout, basically.
That like, I track those as well.

Speaker 4: 00:35:45

Yeah, there's like a lot of competition on those outputs too.

Speaker 1: 00:35:48

But those don't need CPFP, you're only doing RBF since we do C-Gash, single C-Gash anyone can pay, right?

Speaker 4: 00:35:54

That's true, but they still have competition around pin, like you could still potentially pin it, right?
So yes.

Speaker 2: 00:36:01

They can be CPFP if U4 is closed, right?
Because I'm not second level.

Speaker 5: 00:36:04

So last year in New York, I gave kind of the total design where like I at every step, I think this is pen resistant, but it adds extra for the HTLC side of things.
It was adding some extra bytes in the like the benign case.
And so it was kind of like out of hand rejected, I guess.
And that was put as like future work.
So yes, that's out of scope for now.
I do have hope that in future we can fix that, but for now.
Put that as future work.
Put that as future work.

Speaker 2: 00:36:42

Can I ask people about if they're concerned about that fee filter thing that I mentioned, basically?
And I guess I have two questions.
Number one, what do you set your current max anchor fee rate to, if at all?
Like, is there a configured value, basically?
And then also, do you factor in your min-min pool or the fee filter of your peers whenever you're doing fee estimation in order to make sure they can actually propagate fully.
Because we're seeing instances where, like, you know, number one, people aren't sending this value, which we need to change obviously, because they should be automatic.
But then also other instances where, you know, due to sort of like them never seeing certain class, which turns out can lead to fee filter of their peers being higher than their own value, their sort of like, you know, notion of like what can probably get skewed.
And this seems to be causing a lot of force because it costs the network generally, you know, as far as like people either not spending that value or the value is set, but like, they're not factoring in the actual greater fee filter.
And right now, like, you know, we've been purging above like 20 sata buyers and like that for the past few months.

Speaker 4: 00:37:31

Right, If you just run a Bitcoin Core node, you will often not see, you'll often not limit your mempool because your peers are limiting your mempool for you.

Speaker 2: 00:37:40

Exactly.
Right.
So that's the interaction I stumbled upon just looking at some of our nodes, like, oh, what's actually going on?
And, you know, for example, like, you know, I wasn't getting purged at all.
But then all my peers were purging from 25 or even above, basically.

Speaker 4: 00:37:52

Yeah, so we don't use, I think we tried to, but then immediately stopped using the mempool minfee for the channel fee, because you just can't.
You can't use the mempool min fee

Speaker 6: 00:38:04

at all.

Speaker 2: 00:38:04

What we're looking to do is basically use our mempool min fee and then some aggregation of our peers fee filter as well.

Speaker 4: 00:38:10

Why not just look at the longest out fee estimate, like 1,008 or whatever?

Speaker 2: 00:38:18

That'd be far too low to actually even get to the mempool.
And then you can't feedbump because it falls out.
So right now people, you know, so people are just fucked right now.
And unfortunately in some senses, because they were at like 11-cider byte fears and like that, and things just falling out.
So we're basically trying to prevent that now by like factoring in the propagation, you know, fee rate itself.

Speaker 5: 00:38:36

Yeah, I

Speaker 4: 00:38:36

think there's no solution to that.
The other solution to that is sometimes it just works anyway, because a lot of times you'll have a path between you and a miner that has unlimited seismic pools.
And so I've seen many transactions confirm like that.
It's not reliable and

Speaker 2: 00:38:52

I don't know what to do about it.

Speaker 4: 00:38:53

But to answer in terms of like the concrete, like how to set fees, you can't, there's no solution.
Yeah.

Speaker 2: 00:39:00

Yeah, so what do people do today for their Macs, for the Anchor fee rate, right?
Are they still doing update fee as before?
Because we stopped, you know, more, but now we're trying to do something slightly more automated, like, you know, not get into the next block, but instead just get into the mempool, or rather, like, get into a dominant relay path, that is.

Speaker 4: 00:39:16

Yeah, we just use update fee as before with a relatively long estimate time horizon.

Speaker 2: 00:39:24

But do you see the issue in that, like that can result in people's transactions just falling out of the mempool like right now?
It can.

Speaker 4: 00:39:30

It totally can.
I don't know what the solution to that is though.
There's no solution, right?

Speaker 2: 00:39:36

Yeah, yeah.
I guess we've at least identified the problem.

Speaker 4: 00:39:39

Mempool min fee and peer fee filter, maybe you can do that max with the long time horizon.
But even the long time horizon estimate, in my experience is usually better than MIMPL and MINPHI.
It's usually always higher, but I guess, there are certainly cases where that's not the case if MIMPL skyrocket.

Speaker 2: 00:40:02

Yeah, okay, that makes sense.
I guess we'll just try to explore this area because otherwise, like, you know, people ask me what to do and I'm like, I'm sorry, we can't do much right now.
We'd have to like, you know, it needs to stop raining.
That's like literally the response.

Speaker 4: 00:40:16

I mean, you can, if you submit your transactions directly to like mempool space or endo that accepts public connections.

Speaker 2: 00:40:21

Well and so that's the other thing.
It does usually work.
Yeah I've been trying to get them to basically you know do their fee accelerator thing but use it but like keysend based basically.
So I keysend you with a txid.
Nothing without

Speaker 4: 00:40:32

without any accelerator or anything if you just send those transactions, it does usually work.
Because there are enough miners with unlimited mempools that like within a day you'll get mined.

Speaker 2: 00:40:47

Sure, sure.
I mean, yeah, and at least like the key sending was a way like because people actually people could write some side script to automate that, right?
Like if there is a quote-unquote mempool node or mempool ops base node or whatever, they could key send with a TXID to get a fee bumped versus like logging and doing whatever credit card thing they do today.
So that's it's a tool at least.
Maybe I can, you know, continue to bother them in DMs and I'll write a prototype or something like that for them to do.
But okay, all right.
Curious if anyone had any good solutions.
Seems like there's none, really.
We'll look into whatever fee filter aggregation thing and see if we can alleviate some of the pain until the weather improves.

Speaker 5: 00:41:19

So the last recap is we will need, so we're asking from you, I guess, would be explicit ACK on the v3 side of things when it comes to topology restriction and whatnot, right?
As a general one parent, one child strategy, is that good enough for everyone to move forward and what kind of, what kind of sizes do they

Speaker 4: 00:41:37

think are realistic?
To be clear, if we're doing this on the like templated format, it needs to be one parent, two children.

Speaker 5: 00:41:45

Yeah, well what we do is we would have the child, each child size would be restricted in some way to reduce the pinning, and you RBF it directly.
So technically, it would still be one.
It would just allow the sibling eviction functionality.
We, I think we tried to explain.
Does that make sense?
We'd magically RBF the other child to keep the topology simple.

Speaker 6: 00:42:11

Yeah I think that should be fine.

Speaker 4: 00:42:12

You'd evict the other child if a new child comes in that's better than the other

Speaker 5: 00:42:16

Yes, if it's incentive compatible, yeah.

Speaker 6: 00:42:20

Ah, that

Speaker 4: 00:42:20

is fundamentally incentive uncompatible, but yes that's okay for Lightning.

Speaker 2: 00:42:24

What do

Speaker 5: 00:42:24

you mean?
25 chain limits on the

Speaker 4: 00:42:27

You're fundamentally reducing the value of the mempool.

Speaker 5: 00:42:37

All right, you could turn it off if you want, but...

Speaker 4: 00:42:40

Okay, I mean, whatever.
It is fine.
I was just...

Speaker 5: 00:42:44

We can make better judgments about RBFs in this situation, much better judgments with the symptomology.
So, yeah.
Yeah.

Speaker 4: 00:42:52

I mean, one parent, two children with only confirmed spends seems like it should be practical but maybe it's just not worth it right now, which is fine too.

Speaker 6: 00:43:04

Yeah, I think we can discuss the one-parent two-child, I think.
I think that's not inconceivable.

Speaker 5: 00:43:11

Yeah, I

Speaker 6: 00:43:13

think that means if

Speaker 5: 00:43:13

you package Rbf, you're paying for both of them though.
It's kind of annoying, but we'll talk offline about it.

Speaker 4: 00:43:18

Okay yeah and it's also reasonable to just say it's not worth it and we'll figure it out with cluster mempool like we're just doing a short-term thing for now.

Speaker 2: 00:43:26

Yeah I mean I think we

Speaker 5: 00:43:27

can definitely do stuff that's more incentive compatible going forward.
Top block checks are trivial, that sort of thing.

Speaker 6: 00:43:41

Yeah, Matt, I mean, you say it's not incentive compatible, but if you think about descendant limits to begin with, we have the same problem, right?
Like turning down a transaction because your descendant size is it, is kind of silly, but then if you're gonna allow it and then you evict some unrelated transaction that's also valid, you say, well, that's not set-up compatible, but we have to do something, right?

Speaker 4: 00:43:57

Oh, of course.
I'm just Doing it as a threshold of two transactions just feels weird.

Speaker 5: 00:44:06

Too used to 25, Matt.

Speaker 4: 00:44:12

Yes, I've been spoiled.

Speaker 3: 00:44:24

Thanks for your attention.
I really appreciate it.

Speaker 2: 00:44:29

Thanks.
Yeah, I'll do my homework.
This time.
Next time in this discussion, I'll be able to have some important opinions.

Speaker 4: 00:44:35

Yeah, I've got homework to do.
And I mean, it sounds like in any case, there's going to be some, at least minor changes on lightning nodes, even if we do this abbreviated version that I suggested.

Speaker 2: 00:44:47

Yeah, I think eventually we'll need some change to support whatever finally is propagated there.

Speaker 5: 00:44:51

I think the big one is just you're going to have to try packages together.

Speaker 2: 00:44:57

What do you mean try packages together?
You mean like what you're bumping?
Like different combinations of what you saw.

Speaker 5: 00:45:01

We dig it offline.
I'll have to draw diagrams for this.

Speaker 2: 00:45:04

I see, I see.
Hey, I mean, we supported on this thing, but we don't want to hijack that.
But okay, I'll look at the delving posts.
Cool, I guess in the 10 minutes, should we just talk about what people are working on right now?
And like, I guess like goals, other goals for this year other than fix all of the mempool issues.
I know.
I know

Speaker 4: 00:45:35

Mark had a discussion he wanted to open around Trampoline.

Speaker 2: 00:45:40

Oh, okay.
Cool.

Speaker 7: 00:45:42

Yeah.
Thanks, Matt.
So Customers have been asking for trampoline and the async payments, at the very least, for outbound trampoline.
And I was looking at the trampoline spec, and the current status of it is that you essentially have a nested onion that is half the size of the standard onion.
So you have a standard onion that is 1300 bytes.
And the trampoline node and its hop payload receives a nested onion that is 650 bytes.
And from there on out, it figures out how to construct the remainder of the path.
Because the hop payload has to contain at least 650 bytes of data, that means that the trampoline node needs to be within the first half of the 1,300-byte package, such that the 650 bytes can be included in full.
And I was wondering if there might be a way to do away with that limitation.
And I had this idea of, rather than having a fully nested onion included in the trampoline node's hop payload, rather having the trampoline node receive a marker saying, hey, you are the trampoline hop.
You need to figure out the path to the next hop.
And then having the trampoline node rewrap an onion, basically adding layers back on top of what it has peeled and just imposing a limitation on how many bytes of additional data it may add, assuming, of course, that that amount of data is standardized for privacy purposes.
That's really it.
That is all there is.
The one consequence of that is that because it now has to rewrap it and has to start not from the zero byte HMAC, but from the custom HMAC, it will create an interruption in the ephemeral session key chain.
And so it will have to tell the node that is the last node that has been inserted as an additional onion layer what the resumption is going to be of the original ephemeral session pubkey.
Now, I think Christian has a bunch of thoughts and probably a bunch of security considerations that I haven't thought of and dangers that this imposes.
So I'm expecting that this is not gonna work, but I would love to hear from you, Christian.

Speaker 0: 00:48:16

No, it's definitely all right.
It's just a rephrasing that I'd like to do.
If you look at it, you essentially get an onion that departs from somewhere else and then reaches your destination.
And what you're trying to do is prepend a number of steps to get to that next hop, right?
Exactly, yes.
So what you're trying to do is rendezvous routing.
And so we can reapply all of the research we've done for rendezvous routing, including a trick that is very similar to yours.
And let's see if we can rephrase it, essentially.
What you're saying is that if the n-th hop is the trampoline, the n minus 1 needs to tell the n-th that it is a trampoline and therefore needs to switch the ephemeral key, right?

Speaker 7: 00:49:07

Right, exactly.
4n plus 1.

Speaker 0: 00:49:10

Exactly.
So what you could do, for example, and that was one of my rendezvous proposals a couple of years ago, was essentially to say, hey, let the nth node actually process twice.
It essentially receives an onion, it unwraps it the usual way with the ephemeral key it got from the previous hop, just like normal.
The previous hop did not know anything special.
But in its payload, it sees just, hey, a switch of an ephemeral key.
So it knows, oh, I need to re-decrypt using this ephemeral key.
And then I actually get the payload that is destined for me.
So it is from a number of bytes.
It is identical to your proposal, but it doesn't require the n minus 1th to be aware of this thing.
There is another trick that I don't know if it applies here, because there was a way to compress an onion by essentially filling the padding inside of the final destination with the fillers of the previous ones, such that essentially at the rendezvous point, you ended up with a whole bunch of zeros in the middle that you could cut out and therefore get a small onion that you could nest into the larger one.
But I think the first proposal is probably the same as yours.
And we also just fixed the idea of having to have essentially two subsequent nodes be aware of being a trampoline, essentially.

Speaker 2: 00:50:47

Can someone state the difference that we see between the idealized rendezvous and trampoline as is today?
Because I understand what happened was something in between, then what I passed happened.
I guess I'm missing the distinction between pure rendezvous, whatever we all thought that was years ago, and like trampoline yesterday.

Speaker 0: 00:51:03

I think so.
The pure rendezvous essentially was saying, hey, you have a small onion, you have an onion that you somehow compressed and that you could fit it inside of the larger onion that you needed.
That is very similar to the current trampoline construction, by the way.
The proposal that Eric made is essentially to get an onion, process one hop, noticing where do I need to send it next, pre-pending a bunch of hops.
So all you have is the outer onion.
You don't have any sort of nesting in between, but you sort of are playing this accordion of essentially unwrapping the overall instructions and then adding places to get to that next hop.

Speaker 7: 00:51:53

One thing I would point out, though, is that if the node that is the trampoline node is node number n, So node n minus 1 does not actually know that the next one is a trampoline node because it is only contained within the hop payload for n, for the trampoline node.
And rather than being a full prepension, what actually happens is that the trampoline node inserts a bunch of layers between n and n plus 1.
So if it has k shifts, then node n plus 1 will become n plus k plus 1, but node n will remain node n.
So because it has to find a route from itself to n plus 1 using those k additional hops.
The other, yeah, but obviously, I wanted to apologize to you, Christian, that when I was thinking about how to improve trampoline, it didn't occur to me to read up the pre-existing, the prior art on rendezvous routing.

Speaker 0: 00:52:55

Honestly, I don't think I ever wrote them down as such.

Speaker 2: 00:53:01

Tibor, do you want to go as far as this pack-size thing?

Speaker 1: 00:53:05

Yeah, so I'd like to zoom out a bit because fundamentally I think that was either there's a misunderstanding or we're actually just redoing the same thing but just bundling it slightly differently because the current proposal for trampoline, which doesn't match the Eclair implementation, so that's why you need to match to look at the spec PR and not the implementation.
Since we put the TLV field that contains the trampoline onion, this can be variable size.
This can be any size at all, and it's the sender that decides on the side of that.
So you can use any number of trampoline nodes in between.
And Rx proposal, it only shifts that kind of one layer up, from being inside a TLV to just being directly in the route, but it actually has exactly the same constraints that this cannot be greater than the size of the route that are between trampoline nodes.
So this constraint is going to be there regardless of how we do it, regardless of whether we do it directly on the onion route or inside the nested onion.
So I don't really see what your proposal, Arik, fixes here.
And I like the separation of constraint that the trampoline onion is actually in a TLV because it's like it actually shows the nesting that and it actually better matches what pathfinding algorithm has are run.
It's actually a simpler implementation and design.
I tried, I implemented your version today and it seems to me that the nested onion version is actually just simpler and does the same thing.
So I'm planning to understand the reasoning behind your proposed change.

Speaker 7: 00:54:45

Yeah, so I do think that the nested onion is definitely simpler than having to wrap it.
There is one constraint though that it removes because whatever the size of the nested onion is, it has to be at least...
So if you have that certain size, it means that the amount of space that you have is, it cannot be closer to the end of the 1300 total bytes that you can have across the entire Sphinx payload than that size of the nested onion.
Whereas if you have the onion getting rewrapped, regardless of how much additional padding you allow or prohibit, the trampoline node can be as late along the path as you like it to be without any additional privacy implications or constraints.
So it removes a small constraint.
I think if the standard for trampoline onions is 650 bytes, then that constraint is gonna have a stronger or bigger consequence than if say, we set it to like a hundred or 150 bytes but...

Speaker 1: 00:56:05

Why do you want to set any constant?
Why don't you want to let it be completely variable size and decide by the sender on a case-by-case basis?

Speaker 7: 00:56:14

Oh well, you do want it to be standardized for privacy purposes, that I think is

Speaker 1: 00:56:22

quite important.
Not really, I don't think so.
I don't see why, to be honest, because it's rather the sender that decides on, if I want to make it look like I could have that many trampoline nodes, I'm just going to choose that size.
And if everyone does a variable size thing and adds some randomize...

Speaker 4: 00:56:45

That fingerprint's the sender though.
That fingerprint's the sender because all the senders are going to have some default.
They're going to fix the value to 1, 2, 3, whatever.
And then if we start revealing that, it's going to fingerprint the sender.

Speaker 1: 00:57:01

What would you do instead of just randomizing it?

Speaker 0: 00:57:02

So I think you should always use the biggest possible inner onion you can to get to your destination.
But I would argue that this constraint is actually a security parameter for us, because every time that we go from trampoline to trampoline, that could be a full 20 hops of another more locked up funds.
So the number of slots inside of the inner onion times the number of slots on the outer onion is the length of the maximum HTLC, not considering CLTV and fee constraints, but you could end up locking up a lot of liquidity using that.
That being said, even if we choose to have an inner onion that is half the size of the outer onion, we still have about 10 hops to get to our next trampoline, and then that trampoline has 10 hops to get to the next trampoline.
I don't think we have a network that has a diameter that is larger than 10.
So I wouldn't I wouldn't try to push the maximum out of out of the length of the route we can get here because it could even be hurting us.
And I think we do have quite a bit of flexibility already.

Speaker 1: 00:58:28

Yeah, but basically I think I'm not sure why we should try to, I'm not sure why we should try to change it because the Onion nested in an Onion is conceptually simpler and from the implementation side is simpler as well.
So I think we should really explain why it's bad in terms of privacy or an alternative would be better before considering changing it to something that requires more code and more complexity.

Speaker 7: 00:59:00

I do think it makes it easier to have multiple trampoline hops, but I just truly want to emphasize that.
Well, I don't think trampoline is really being used a lot.
So I'm personally fine with just going with whatever the simplest implementation is.

Speaker 1: 00:59:22

Yeah, I agree.
I agree.
I think that should be the reasoning, the simplest, and think that is cleanest from an implementation and conceptual point of view.
And to be honest, I think having implementing both, implemented both versions, I think the onion inside an onion is really the simplest one.
But maybe, but you, you have to

Speaker 2: 00:59:53

yeah, do it as well and other people's And other people's opinion are going to be interesting here.

Speaker 1: 01:00:00

Oh, I hadn't seen all the messages from Lalu.
Can you state that?

Speaker 2: 01:00:08

I think the most important one, I guess, is working backwards from some of the new goals that they have as far as revisiting it.
To my understanding, it's basically like ASIC retrend for mobile users.
So Basically, they can start a send and then go offline, and therefore, Trampoline Hub is basically doing the thing, which improves UX.
To my understanding, that's the main thing that they're working backwards here to achieve UX-wise.
I think we talked about this in New York, T-BAS.
I think you convinced me then, but maybe I've forgotten my prior insight around compatibility with PTLCs. Because at first I thought there was some other thing, or I think it was related with the other routing thing, like the two-phase payment thing basically, where that's slightly different.
But I think that's the goal here, at least with async send retry thing.

Speaker 4: 01:00:52

That's the main goal, yeah.

Speaker 2: 01:00:56

Yeah, and then my other questions was just around, I guess I'm not fully, I need to catch up here.
I guess like understanding like how they can pre-pend and like or they can can they derive a shared secret the entire thing's authenticated and so forth.
Because like my current mental model is similar to what Sebastian talked about as far as like just putting it in the nested TLV.
But I missed the thing around how the pre-pend works.
I'm not sure if that's like written down anywhere in any of the spec PRs or if that was just something that was, you know, unveiled just now.

Speaker 6: 01:01:22

Yeah, I can, I can,

Speaker 2: 01:01:23

I can

Speaker 4: 01:01:24

figure it out?

Speaker 1: 01:01:24

Yeah, it isn't yet, but it's actually very similar to the switch fMRI key idea that Christian described a long time ago.
I think it's probably even still in the wiki part of the github and was discussed maybe even in Adelaide or something like that, where there was this proposal that we discussed that Christian detailed how you would do that switchfkey.
I'll try to find it but it's very similar to what Arik is doing right now.

Speaker 0: 01:01:53

I should really just write these trickeries up, because sometimes they can be useful.
Onion messages, for example, would be really nice to have rendezvous for.

Speaker 2: 01:02:03

OK, or you mean rendezvous onion messages?
Or use it to make rendezvous better?

Speaker 0: 01:02:10

You could add rendezvous to onion messages to essentially talk to some hidden service.

Speaker 2: 01:02:18

Ah, sure.
Hey, here we go.

Speaker 0: 01:02:23

That's where that research comes from.

Speaker 2: 01:02:27

No, yeah, no.
I guess we can just have our websites in lightning pretty soon.
Quick thing on the side.
I remember last time, I think, Arki said you're going to start to look at taproot gossip stuff.
I'm assuming now you're full on trampoline.

Speaker 7: 01:02:46

Trampoline, it really depends.
If we just go with the current proposal that, I mean, not the current proposal, the current status quo that is already implemented.
I have a functional PR, so that completely matches.

Speaker 2: 01:03:03

A PR for Trampoline?

Speaker 7: 01:03:04

Yes, implementation, yeah, for trampoline.
So once that is done, I would hope I can switch back to taproot stuff quickly.

Speaker 2: 01:03:14

Cool.
And then as far as this, if y'all like searching about stuff as far, because I don't know if this was like implemented in the past, because you know, there's only a few of them, but like, Trampoline node discovery, would it, and also like, you know, for example, like if a new node is bootstrapping basically, and the only one in 16 Trampoline nodes, is there some new query gossip type extension thingy, where we sort of like tell them we only care about this feature bit in the node that we get, we got to think about something like that or something else entirely.
And this is basically like, okay, a new node joining, they're not going to get the entire graph anymore and assuming template nodes have a feature bit, do they have some queries essentially to basically only fetch those nodes in order to like do, you know, bootstrap and do pathfinding.
So basically, what does bootstrapping look like for a new node?
I don't

Speaker 4: 01:03:53

think there's, I mean, currently nodes can just download the whole network graph.
So probably we'll just keep doing that.

Speaker 2: 01:03:59

Okay.
To

Speaker 4: 01:04:00

add feature bits so you can, but I mean it depends on the nodes, depends on the LSP, you know.
Some people will do the like async approach of just using one LSP and sending everything to them.
Some people will have a few more hops in the trampoline for privacy.

Speaker 2: 01:04:17

And this is Trampoline for Async Sense, not necessarily restricted channel graph.
Because you feel the channel graph is so small, you don't need to worry about it.

Speaker 4: 01:04:26

Currently the channel graph is sufficiently small, no one really seems to have a problem with downloading the whole thing.

Speaker 1: 01:04:35

And actually I want to highlight that there are two separate steps in the trampoline design.
What we are working on right now with Arik is finalizing mostly the Onion construction.
And it's quite independent to how we then decide to potentially advertise trampoline nodes and then decide how to advertise the rates and LLs. So I think those two can be phased separately because only only having the Onion construction is useful for wallets because in wallets you don't have any issue.
You can just include those in the invoice and you can just either directly connect to LSPs that you know of trampoline stuff or find a small route to them.
And then we can face the potential advertisement and advertisement of fees in another future afterwards.
So that's why I think the...

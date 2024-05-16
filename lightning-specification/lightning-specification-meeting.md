---
title: "Lightning Specification Meeting"
transcript_by: carlaKC via tstbtc v1.0.0 --needs-review
tags: ['lightning']
speakers: []
categories: []
date: 2024-05-06
---
Speaker 0: 00:00:03

All right, should we start now?
Okay, Rusty, does someone know if Rusty, or maybe Rusty is traveling back from Austin and will not be attending?
Does anyone know?

Speaker 1: 00:00:15

I believe that's correct.

Speaker 0: 00:00:16

Okay, so maybe we'll keep the first item because it's a PR he opened and I'm just waiting for him to apply some changes.
Unless someone has comments on the implementation of the cleanup, spec cleanup about the unused features.
I don't think so, but if anyone has any issue with it, I can say so now.
Yeah, but Let's just skip to the next one.
So the next one is go see 12 blocks delay channel close follow up.
So this is something we already merged something to a spec saying that whenever you see a channel closing up on chain, you see a channel being spent on chain, you should wait for 12 blocks to allow splicing to happen.
And it's just one line that was missing in the requirement.
And then they found in the older splicing PR.
So there's already two acts, three acts.
So is anyone opposed to just merging that?

Speaker 2: 00:01:16

Oh, look, it looks like I acted.
I don't remember acting this.
Maybe GitHub made that up.
I don't know.

Speaker 0: 00:01:25

Perfect.
So unless someone screams before tomorrow, I'll just merge that one tomorrow.
Then the next one is constant size failure on decryption.
I understand that this was a topic that was covered in the last spec meeting but it doesn't seem like it's merged or removed.
So

Speaker 2: 00:01:45

I think there was some I think there was a little bit of agreement last spec meeting to suggest that this be rewritten as a more general, you should try to avoid being fingerprintable as to whether or not you were the sender based on externally visible timing and that kind of thing, rather than trying to be super specific here.
But it looks like that didn't happen.

Speaker 0: 00:02:11

OK, then that sounds reasonable.
So we should not merge it as is right now.
And we should just wait for it to be updated and more general.
OK.
Sounds good.

Speaker 2: 00:02:20

I might be making that up, but I think that's what I remember from two weeks ago.

Speaker 1: 00:02:25

OK.
That's what I remember.

Speaker 0: 00:02:28

OK.
So I'll add a comment directly on the PR so that the author can update it.
All right, so the next one is an update to a notation for math content.
Oh, perfect, right on time.
This is, We're talking about 1158, which is a follow up on the PR you mentioned about using math content notation.
And there are some things that are not rendering correctly.
So someone came and started fixing them.
So could you Just have a look at it and hack it if it looks good to you.

Speaker 1: 00:03:06

I'm sure.
What's the number?

Speaker 0: 00:03:09

1158.
1158.

Speaker 1: 00:03:16

Oh, cool.
OK.

Speaker 0: 00:03:17

All right.
So just have a quick look at it.
Some of the things were not at least for me were not rendering correctly in the markdown.
Now they are.
There's only one remaining.
But apart from that, it looks like it's okay.

Speaker 2: 00:03:30

Yeah, I

Speaker 1: 00:03:30

can have a look at this.

Speaker 0: 00:03:32

Even though I find it quite a bit more painful to write, do you still write it?
Because seeing at this reviewing it, reviewing the diff in plain text is horrible.
So you have to just look at the render thing and it's kind of hard jumping between one and the other.
Was it hard writing it down?

Speaker 1: 00:03:50

The thing is, I think for those that have written a lot of LaTeX, you basically compile it with your eyes.
You know what I mean?
But I think if you're not used to all of the random dollar signs and stuff like that, it is a little bit more difficult.
But yeah, that is somewhat of a trade-off, I guess, in that like, you know, once you're used to it, you can look at this and know if it's doing the right thing, but otherwise, you know, like you need to just sort of rely on the rendered version.
Yeah, I can definitely, I

Speaker 3: 00:04:14

can definitely answer your question.
I think it makes the spec itself easier to read and it makes the diffs way harder to read.

Speaker 0: 00:04:21

Exactly.
So that's probably a good trade-off.
More people read the spec than review it.

Speaker 1: 00:04:27

Yeah, and for this one, I think it's still stuff where it's like, you need to basically do dollar sign back tick basically.
In some areas I did back tick dollar sign, because back tick is like a reflex for doing any like markdown code stuff anyway.
So.

Speaker 0: 00:04:42

Okay.
All right.
Sounds good.
So just have a look at it and once you hack it and the last comments are fixed, I guess we can merge it.

Speaker 1: 00:04:51

Sounds easy.

Speaker 0: 00:04:52

Yeah.
So the next step is Matt making the channel update in Onion Errors optional.
It's something we've been talking about a lot, but no one really bothered to open a spec PR because it's just a chore and it's annoying, but I think we should definitely do it and this is already happening on the network.

Speaker 1: 00:05:11

And what's, what's the rationale here?
I guess you're saying that like fingerprint vulnerabilities, like when they shouldn't broadcast it, It was like a private channel or something?

Speaker 2: 00:05:22

Yeah.
No. Previously, Rusty, like six months ago or something, and several spec calls in a row, Rusty pointed out that if you receive an HTLC and then you send back a unique channel update to that peer, then it will be in their gossip store and then you can query them for whether it's in their gossip store and then you can identify who sent that specific HTLC.
This updates the spec to say, you must ignore it if you see it, and then also you should no longer require that it be there with the intention that nodes will eventually just start removing it.

Speaker 1: 00:06:08

But if you ignore it, that like sort of like, I mean, it does have a purpose, right?
The purpose is basically giving you the most up-to-date channel update.
Whereas in the past, like, you know, this basically lets you not have to sync all of Gossip because you get the new stuff that you need when you try to, and you feel all right.

Speaker 2: 00:06:23

No. So nodes, you should never be relying on this.
Frankly, nothing, the update in the onion error should never have been used to begin with because if you send a new Fiore, then you shouldn't be applying it for quite some time.
I think, I forget what the recommendation is, but it's like a while.
It's like tens of minutes or something.
And by that point, everyone should

Speaker 1: 00:06:51

have- Does anyone actually do that?
We, we instantly- Yeah, absolutely.
Okay, well, I mean, so what we do, so we shift that to happen basically, we'll do a thing where like, we'll give an individual another chance, basically sends a new update before saying, okay, well, I think it's whack or not doing it.
But I do think it has a role.
The role is that you don't need to sing a lot of gossip.
You can just basically get the stuff at runtime, but I realized this privacy issue here.

Speaker 2: 00:07:16

I mean, getting in a runtime is also really shitty like people don't like payments that are really slow.

Speaker 1: 00:07:23

Well I guess what I mean is that like the thing what the difference is that like if you don't apply this on runtime you could you're potentially permanently failing a route because you don't have the latest thing, right?
So it's actually going to help to improve payment UX, right?
Because now you can get the new value and try versus just saying, I'm giving up until gossip happens, which can be hours, right?

Speaker 2: 00:07:42

Well, gossip's not hours.
What?

Speaker 1: 00:07:44

I mean, just depending, right?
I mean, I don't know if anyone's measured, you know, the propagation speed, but that's just to say that

Speaker 2: 00:07:50

like, people have,

Speaker 1: 00:07:51

it is a UX optimization.
It is a UX optimization basically.
Cause otherwise, like if you don't get this new value, I mean, I think it matters more for, but do you, What

Speaker 2: 00:08:00

do you do with the value currently?
Do you actually apply it to your Gossip Store so you're vulnerable to the fingerprinting?

Speaker 1: 00:08:06

Yeah, so today, well, no.
So we won't apply it to Gossip Store, we'll apply it to that session basically.
So we'll apply it in memory, that's to say.
So we'll get the new thing, we'll buy it in memory, we'll try it again, and then we have something called second-hand logic, which I think we took out of now, but otherwise, we will wait for that thing to come back again, but right now we'll apply it in memory, which is useful because then we can just try again.

Speaker 2: 00:08:27

To be clear, you apply it just for that payment session, right,

Speaker 1: 00:08:30

So just for

Speaker 2: 00:08:31

that payment, you'll recalculate the route.
And if you happen to decide to use the same channel, you'll try that one.

Speaker 1: 00:08:38

Yep.
And many times, you do actually retry again successfully because this is the thing that you're missing.
So I see why we should discourage it, but I think removing it altogether would hamper UX.

Speaker 2: 00:08:53

So it's worth pointing out that you have the same, that becomes a privacy issue with PTLCs because you don't want to indicate to someone that you're actually retrying the same HTLC, potentially.
And then separately I think, I'm curious if you could go take a look at cases where this actually kicks in, I imagine it's basically just right on startup, right?
Like the only time you're ever actually out of sync with gossip enough that someone's sending a new gossip update causes payments to fail should just be on startup.

Speaker 1: 00:09:31

Well, I think it's different, right?
I think your model is that they're using RGS to basically always get the latest gossip when they come up as a mobile node.
But our model is instead that, oh, they'll get the latest update when they actually need to do so.
Because we don't really aggressively think of the gossip network, and we don't have a built-in RGS type thing.
You know, being bundled into the-

Speaker 2: 00:09:50

So you mean specifically on mobile nodes,

Speaker 1: 00:09:52

not-

Speaker 2: 00:09:53

Well, yeah.

Speaker 1: 00:09:53

So you're thinking about

Speaker 2: 00:09:54

this on mobile, not on-

Speaker 1: 00:09:56

Yeah, exactly.
I think on server, like you're online enough that you should have the latest stuff like you're saying, but I think on mobile it's a bit different if you're not using like a fast gossip, you know, download type of thing.
And assuming that you have all the latest.

Speaker 2: 00:10:07

So that affects what, just Zeus?
Is that the only?

Speaker 1: 00:10:11

I guess Zeus, Breeze, Blix, whatever the other ones.
Breeze is moving off.

Speaker 2: 00:10:17

They've been

Speaker 1: 00:10:17

saying that for a while, but they're close to her, I guess.

Speaker 2: 00:10:20

Yeah, yeah, they have been saying that

Speaker 1: 00:10:21

for a while.
I remember when Blue Wallet moved out, but then they don't really exist anywhere, right?

Speaker 4: 00:10:25

Right.

Speaker 0: 00:10:29

What was

Speaker 2: 00:10:29

I going to say?
Yeah, so maybe those wallets just need to, maybe we should have a conversation with those wallets about doing something more intelligent with Gossip.
Because this can cause lots of additional payment latency for them, right?

Speaker 1: 00:10:45

Well, I think the difference is that obviously, syncing the gotcha every single time does add additional costs to the node or to the implementation basically.
But I think we do have a lot of security UX trade-offs in the protocol generally.
This is yet another one basically.
I think we should think about, do we really want to just disallow this altogether, which I think was a useful thing for particular payment latency, or advise implementations what to do about it?
And can we mitigate that fingerprinting thing somewhat?
Like if we're sort of like, you know.

Speaker 2: 00:11:15

It seems like the only thing you can do is either...
So it seems like we agree that this is totally not useful for non-mobile nodes, and if we had no mobile nodes relying on peer-to-peer gossip, we would just drop this entirely.
So given that, for mobile nodes, if you're particularly behind on gossip, it seems like you're just going to have a lot of payments that are going to hit this all the time.
And you're just going to have a ton of round trips, which doesn't seem super great either, but maybe you hit this once while you're waiting for gossip to sink.
It's not clear to me what the...
You're telling me that there's these mobile nodes out there that start up, don't really sink gossip, send the payment.
Well, it's not that...
One channel on their path.

Speaker 1: 00:12:05

Well, so the thing is they'll get the channel updates, but they're not trying to sink whatever 50,000 gossip changes that may have happened while they're offline, right?
I think,

Speaker 2: 00:12:13

I think you

Speaker 1: 00:12:13

should also compare it to, like, they may already encounter a failure due to whatever other issues basically.
Which one dominates?
Is it going to be the failure due to them not having latest update or just everything else that can go wrong along the way basically?
So it's like one of the two things.
My intuition is that the latter, the second one is basically what matters more, basically just random failures that happen versus this particular case.
Because I'm assuming like people are going through the similar nodes anyway, right?
And if they have those set of like, you know, 20 updates versus out of the 50,000, like they have a pretty good experience.

Speaker 3: 00:12:48

Yeah, I think it comes down to how fast you can sync the unsynced gossip.

Speaker 1: 00:12:54

If you have like one of these third party servers, it's very fast.
If you don't, it takes more time.
Because also like, well, I mean, so things like in the past, people would do the complete backlog, which wasn't great basically because you would just send all this data over the entire time and try to validate it all.
Now, we do have the go back 24 hours or whatever, but that doesn't cover everything.
If you want to cover everything, you either need to do that all the time, or do one of these rapid gossip thingies, or hand-wave mini-sketch something.

Speaker 2: 00:13:23

The...
Right, but if you go back, if you go back a ways, you should hit everything basically.
Yes.
I'm confused why you have this problem to begin with.

Speaker 1: 00:13:42

Well, I mean, yeah.
So the thing is, we stopped doing the backlog altogether.
In the beginning, everyone did the backlog the entire time, you just sync the entire graph whenever you reconnect.
But now, the thing is, if you're reconnecting every single time, you're getting that particular backlog.
We just eliminated it because it was just flappy, and it would just cause a bunch of CPU, and it was just unnecessary.
What we do is, we basically have three or eight peers or whatever.
We only enable active gossip on those three peers basically.
But we will periodically do the full channel range thing to basically spot check, do we miss any new channels?
But otherwise, when we reconnect, we're not asking for any sort of backlog at all, basically.

Speaker 2: 00:14:20

When you reconnect or when you start up?

Speaker 1: 00:14:24

Both.
They're basically the same for us in this context.

Speaker 2: 00:14:27

That's why Gossip's unreliable.
Okay.

Speaker 1: 00:14:32

Well, I mean, the thing is, you're saying for the routing nodes, they're gonna be getting those messages anyway, because you also are retransmitting your own messages, but also there is rate limiting that everyone does as well, right?
But the thing is like, you know, I guess a question for you all, like, so then If you're not using RGS, what's the backlog that you ask for?

Speaker 2: 00:14:50

And I guess for Phoenix

Speaker 4: 00:14:51

as well.

Speaker 2: 00:14:52

On startup, the first three peers we talk to, we do a full graph sync.
And then we just listen to the live updates from everybody.

Speaker 1: 00:15:02

Isn't that the same way I just described?
So, full graphic is using the channel height range, basically, and not doing a backlog for, you know...

Speaker 2: 00:15:10

No, including backlog.
Like, we download everything.

Speaker 1: 00:15:13

Oh, so you redownload the entire channel.

Speaker 2: 00:15:16

The first three peers we talk to.
Because you have to if you actually care, if a node generates a gossip update that has a timestamp that's a little old, that's the only way to make sure you get it.
Because the gossip sync stuff uses the timestamp and the message rather than the timestamp you received it.

Speaker 1: 00:15:33

Yeah, I mean, and I think that's where our model differs.
Like, on Startup, you download the entire graph.
We say we don't need to download it because if we're using that particular edge, we'll get this new update and apply it and can just focus on the channel that we use versus the 90% that we'll never touch.

Speaker 2: 00:15:46

But that means anytime you hit this channel again, like if there's some channel that you happen to not get an update for, that means every time you start off, like every time you try to send a payment, you're going to hit the first payment attempt is going to fail and you're going to get, you're going to have to retry it.

Speaker 1: 00:16:04

Well, that's either like, if we hear about

Speaker 2: 00:16:06

it or not.
What?

Speaker 1: 00:16:09

I mean, the question is like, you know, do we hear about it later on or not basically, right?

Speaker 2: 00:16:14

Sure, but you were telling me, You were telling me you were mostly worried about this for mobile nodes, and they're not going to hear about it later.
They're not online long enough to hear about anything.
Well, I

Speaker 1: 00:16:22

guess what I'm trying to say is that this is a class of failure that can happen.
Including the channel update, you basically allow individuals to recover from this type of failure, whereas otherwise they wouldn't be able to recover at all.
There are other types of failures that can happen.
Not being offline, channels being

Speaker 2: 00:16:33

down, disabled.
Only kind of, right?
I thought we agreed that this is not the kind of failure we expect for any node that's online all the time, like any regular merging or whatever.

Speaker 1: 00:16:44

Yeah, It's for nodes that aren't aggressively syncing channel updates, basically.

Speaker 2: 00:16:50

Well, I thought we agreed that this is mostly like mobile node setups, right?

Speaker 1: 00:16:55

Yeah, and to me, a mobile node is a node that doesn't necessarily care about syncing aggressively syncing channel updates, basically.
Because they're only going through a small portion of the network anyway, right?
So why sync everything?
That's my view at least.

Speaker 2: 00:17:06

No, I mean, fine.
But that – so my understanding of your model is they're not going to sync anything, Right?
They're going to sync

Speaker 1: 00:17:16

stuff as it comes in.
We sync new channels, yeah.
Yeah, we sync new channels and then get the rest as it comes in.
And then on demand, if there's something that we never heard of that is still new, we'll hear about it through the error.

Speaker 2: 00:17:28

If somebody tries to pay somebody regularly, they're just going to hit this every single time they try to pay them, right?

Speaker 1: 00:17:35

Not necessarily.
Why would that be the case?
If they hear

Speaker 0: 00:17:37

about it.

Speaker 3: 00:17:37

Would a new edge be applied?
Wouldn't a new edge be applied when it like actually hears about it?

Speaker 2: 00:17:44

Well, it's a mobile node, right?
It's like online for a minute at a time once a day.
So it never is going to really see stuff for the most part.

Speaker 3: 00:17:54

And we don't currently commit the edge to the actual graph because of this attack, right?

Speaker 1: 00:17:59

I checked last time, and it's just in the session.
I can check again.

Speaker 2: 00:18:05

But I

Speaker 3: 00:18:06

guess- I think that's right, is that we would hit it every single time.

Speaker 2: 00:18:11

I'm going to do some

Speaker 1: 00:18:12

loud, clacky stuff now.
But I guess, T-Best, How's it clear into this?
I'm guessing, do you guys do some RGS type of thing, basically?
Or I guess because it's trampoline, they don't need to worry about that anyway?

Speaker 0: 00:18:23

Exactly, for Phoenix, since we use trampoline, we don't worry about gossip at all.
We don't do any gossip.

Speaker 2: 00:18:32

I didn't

Speaker 0: 00:18:32

feel like that.
I thought Zoo shipped some

Speaker 2: 00:18:34

kind of other gossip scheme that was not peer-to-peer because they had some issues with the peer-to-peer stuff being so slow.

Speaker 1: 00:18:41

Correct.
I think what they do, I think Blix does it as well, where basically they'll do like a fast channel graph import to Bootstrap, but I don't think they spot check.
So I think they let people basically skip downloading from P2P because they download like a snapshot that goes like directly into the DB.
But I don't think they do like a on startup spot check, let me see what the new channel graph thingy is.

Speaker 2: 00:19:00

Right.
So they're basically doing RGS, but via different format.

Speaker 1: 00:19:05

They're doing RGS, but only for initial bootstrap.
They don't do a spot check.

Speaker 2: 00:19:08

Right?
So you'll have...
Do they not redownload?
Like if they started the wallet, like they installed the wallet six months ago, they've opened it once a week for a minute, never really done any peer-to-peer graph sync.
Do they like re-download it at some point or do they just like let it get progressively more and more stale?

Speaker 1: 00:19:26

I don't think they re-downloaded it, but I'd have to check.
I know, I'm pretty sure Blix doesn't re-download it for Zeus.
Maybe they do, but I know like they were collaborating on like, you know, a fast channel graph loading need that was somewhat L&D specific.

Speaker 2: 00:19:41

All right, so how about I chat with Evan at some point then, and then we come back to this next meeting.
Because it seems like, you know, if Zeus and Blixter are doing something else, then we don't really need to care about this and we can drop this.
If they're not doing something else, then we can revisit whether this makes sense.

Speaker 0: 00:20:01

Anyway, if we take a step back, it at least makes sense to update the spec to say that the channel update is not mandatory anymore but optional because it's already the case that some nodes just do not include it because of plugins on different implementation.
I think it was- Or

Speaker 2: 00:20:19

Lightning has had a bug for a while that they don't include it.
I don't remember why.
I don't think I ever got debugged.

Speaker 1: 00:20:25

But I think that makes sense too, Max, right?
And that like, I think we do it in two steps.
We can like say it's optional now and then remove it later.
And one, we talk to the other one, it's like, OK, well, it's already optional as is.

Speaker 0: 00:20:37

Yeah.
And we should really say that it's optional because it's what actually happens on the network right now.
So people need to allow reading such an amount that doesn't contain a channel update.

Speaker 2: 00:20:47

Well, there's nothing in this PR right now at all that says anything about suggesting not including it.
It just says that you have to ignore it if it's there.
You know, LND can choose to do something else for its payment sessions, but.

Speaker 1: 00:21:00

No, the thing is, it does remove the section on, should you use the new channel update or not.
Right?
If you look at the line for Fortuno

Speaker 2: 00:21:09

2, it's saying you don't use it.
The section on using the new channel update is like, it says you should broadcast it and you should apply it generically, which you definitely should not do.
I don't think anyone does anymore.

Speaker 0: 00:21:21

Yeah, we changed that.

Speaker 1: 00:21:26

I mean, I guess we should see what CLM does here as well.
But it seems like we can at least do the language to stamp it that's optional and then look at this later.
Because I think clearly it's like a private security trade-off, but we should just make sure, like making conscious decision here and not degrading UX in this way.
We've done this for a long time now and we haven't had any complaints about it either.

Speaker 2: 00:21:47

I'll share with Evan and see what he has to say about what Zeus currently does and whether this impacts what they do.
And if he says it doesn't impact them at all and he doesn't think it impacts Blix, it sounds like we would all probably be on the same page that we should just go ahead and drop it.

Speaker 0: 00:22:16

All right, should we move on to Ball 12?
Okay, so in Ball 12, there were some latest changes to the specification, mostly allowing some fields to become optional that were previously mandatory.
And I think there's still an ongoing discussion about the case where you specify both an offhand ID and blinded paths.
And I'm quite not sure what everyone's opinion is because it looks like people mostly agree with each other, but it's unclear.
I'm linking the comment right now.

Speaker 4: 00:22:52

I don't think Rusty agreed with us on it, actually.

Speaker 0: 00:22:56

You don't think that what?

Speaker 4: 00:22:58

Rusty agreed.
I think he wanted to have...
So we wanted to support sending, but not necessarily receiving.
But if we do not have the node ID when there's a path, we would have to support receiving as well.
Okay.
I think most of that discussion occurred on the spec meeting notes from last time, and Rusty may have not seen all of that.
At least it was implied that we would support sending, not receiving.

Speaker 0: 00:23:31

Okay, so we should just ping Rusty and wait for him to continue the discussion directly in the comment.

Speaker 2: 00:23:39

Yeah, sadly we're going to ship this as a release like tomorrow.
So without Rusty here it's a little tricky, but we're probably just going to ship it as is and it should at least work with Phoenix, because it at least matches what Thomas' understanding or Thomas' goal was.
So well, Core Lightning will figure it out or we'll fix that later.

Speaker 0: 00:24:09

Okay.
And on our side, we can still iterate a lot.
We haven't skipped anything and we can update anything based on the how the cross-compact test go.

Speaker 5: 00:24:19

Well, sorry, sorry if I jump in.
What is the problem with, from the CLN point of view, that I missed the last two spec meeting?
There is someone that can be, can do a summary.

Speaker 2: 00:24:32

I don't think this has a problem from the CLN point of view.
I don't, given this was a change we only made two weeks ago, I kind of doubt Rusty has gotten to implementing any of this in Core Lightning.
It's more of a disagreement on what we should do.

Speaker 5: 00:24:51

OK, yeah, because I was also trying to grab the Bolt12 specification to see What are the missing points from the correlating side to try to rebase our current implementation on the spec meeting, on the current spec?
So because I see that we missed something, But I still need to figure out what.

Speaker 0: 00:25:17

Yeah, I think there's a lot that changed since the latest implementation was done in CLN.
And Rusty said that he wanted to allocate some time in May to work on it and make sure that CLN was up to date with the spec.
So I guess that once he's back from OSD and he's done with Covenants, I think he's going to be working on that.
So that should help move the needle a little bit.

Speaker 5: 00:25:40

Okay, makes sense.
Thanks.

Speaker 2: 00:25:45

There's also this issue where LDK and Core Lightning disagree on how you should build a reply path for an invoice request message.
Rusty indicated he was going to acquiesce and do or at least support what LDK did, but yeah.

Speaker 0: 00:26:03

Can you detail?
I missed that issue.

Speaker 2: 00:26:07

LDK sends an invoice request with a reply path that is just like any other blinded path it would make, or a blinded onion message path that it would make.
But Core Lightning, when Core Lightning sends an invoice request, it builds a reply path directly back along the path it sent the invoice request.
So like, for example, if they have to do a direct connection to an introduction point to make an invoice request, they'll just send you a reply path that is that introduction point straight back to them because they currently have a live connection to them, even if they don't have a channel with them.
And so Core Lightning doesn't have any logic to do direct connections for invoice request replies only for invoice request sends.
And Rusty indicated they were just going to add support for that for replies.

Speaker 0: 00:27:06

OK.

Speaker 1: 00:27:08

Can you repeat that last part?
I don't know if I said it.
You're saying that they don't do direct connect for invoice replies.
I thought the reply is going over the connect that the sender made?

Speaker 2: 00:27:20

So if we send an inverse request message to core lightning, we will include a reply path that is just like we would do for any other reply path.
So it might, so like, you know, I might connect via the async node to connect to the introduction point for requesting a invoice from Rusty, but the reply path I include with that is just gonna be, you know, whatever LDK's router decides, and that may, might be like, you know, you must reply via, I don't know, Blitz node or whatever.
And then there on the receiving end, Core Lightning will say like, ah, well I'm not connected to them, I can't reply.
And they were going to add connection logic for that at least while yeah.

Speaker 1: 00:28:09

Okay that makes sense.

Speaker 0: 00:28:14

But either connection logic or just routing logic because they could also just find a path to a BLEEX node from themselves to send a reply right?

Speaker 2: 00:28:22

Right, right, routed or connection.
I mean either way they were going to add support for making sure that message gets there because currently it never does.

Speaker 0: 00:28:31

Okay.

Speaker 2: 00:28:31

So generally when people have been trying to test Bolt 12, they've hit this issue.
This seems to be the low hanging fruit that everyone's been hitting.

Speaker 0: 00:28:42

Okay, sounds good.
So it means cross-compat tests are making progress and people are starting to experiment with the end-to-end flow, which is really nice.

Speaker 2: 00:28:52

Yeah, and people seem to be having a lot of success with the end-to-end flow, at least for LNDK and LDK or CoreLightning to CoreLightning.
I don't know that there's been a lot.
I think there's been some developer testing with Eclair, but not kind of general hackathon testing.

Speaker 6: 00:29:14

Tom from Strikes got Eclair LNDK the other day, and then also ran into the same bug with sea lightning so that's also working.

Speaker 2: 00:29:22

Oh right right yeah so anyway that that's it's making good progress I haven't been able to get in touch with Fabrice.
He's off this week.
So I haven't made the progress that I wanted to do for our official cross-compat test, so we can merge the spec.
But hopefully next week or whenever, Fabrice will get back and we'll be able to work on that.

Speaker 0: 00:29:49

Okay, and I think we'll need to wait for Rusty to update CI as well and to make sure that he does nothing that he wants to change on the latest changes we've proposed.
But I'm hoping that May will be the month where we can finalize that.
All right, so kind of related to offers, Val had a comment for the of blended paths and trampoline, but I think I addressed it in by directly answering the comment.
Val, does that make sense?

Speaker 2: 00:30:19

VALERIE SCHMIDT-CAMPBELLO

Speaker 3: 00:30:20

Yes, that does make sense.
Thank you for clarifying that last part.
One thing I'm wondering about that for, we don't have to get into it now, but for async payments, it looks like the sender will be setting the final CLTV.
And with acing payments, you want to set a quite long CLTV.
So we were hoping the first trampoline hop could set the CLTV to a more reasonable one.
I don't, I have to think about it more but that was something that came up.

Speaker 0: 00:30:53

Yeah okay, yeah we'll have to think about this because in that case you're right that maybe we'd need another mechanism than letting the sender directly send it in the trampoline onion.
So we need to think about that.

Speaker 2: 00:31:06

Yeah, yeah.

Speaker 0: 00:31:08

Okay, okay.
Let's keep discussing it directly on the PRs. All right, so next up, quiescence.
There has been some progress on quiescence.
We investigated with Kegan adding the go on message at the end.
But unfortunately, what we realized that is that really whenever you do a fundamental protocol, there's going to be two stages.
One stage where you are still doing things that you can just roll back and ignore.
And one stage where you just cannot abort anymore because, for example, you gave a signature for the current funding transaction, but the protocol is not complete yet because usually you need to wait for the other guy to also send signatures.
And at that point, you cannot roll back anymore, and you have to handle any kind of failure in the protocol, disconnections, and everything.
And that creates inherently a layering violation that can only easily be resolved by integrating with a channel reestablished mechanism and adding the GAN message on top adds too much complexity, additional complexity, so we figured out that it's really easier to just stick with what we have now without this explicit termination of quiescence and tying the termination of the inner protocol or the disconnection to the termination of the quiescent states.
So I think we should be able to resume then our cross compatibility tests with the existing code base.
Is that correct, Kegan?

Speaker 3: 00:32:36

Yeah.
As it stands right now, L&D is ready to do cross compat testing with the existing spec with Eclair and I guess whoever else has implemented it as is.
They're, in my opinion, still need to be spec updates, but these are more like clarifying positions of basically the current design, not like any changes to the fundamental approach that we're going.

Speaker 1: 00:32:58

I have a question for that.
Like, so If there's not an explicit termination, then how can, in theory, other high-level stuff all using STFU coordinate?
I guess it's sort of like assume it's an internal implementation thing.

Speaker 3: 00:33:09

You have to reacquire a quiescence.
To explain in more detail what's the problem is that for all of these fundamental updates, there comes a point where you get to what we call like half committed, which means that you can't abort the protocol anymore, but only one side has exchanged like some sort of secret data that can no longer be revoked, but the other side has not.
Right now, the way, you can always end up in half committed states during any disconnect process, but we handle disconnect processes cleanly during this channel reestablish process.
If we're at to add this go on message, And we essentially sort of like nuke the inner protocol state.
Then we can end up in a half committed state, but without having disconnected from the peer.
And so you have to ask them the question, how do we resolve this half committed state?
Do we put some sort of channel reestablishment flow mid protocol?
Or like, what do we do?
And it turns out it's like okay, we could try to you know put some sort of like state synchronization protocol downstream of one of these events, but it's like Why go through the extra effort?
We should just instead centralize all of this synchronization through the channel reestablishment process And then essentially force a disconnect if any of this happens and the problem is that the go on message would explicitly Bless a message sequence that can result in this half committed state, and we don't want to do that.
So like I've changed my mind about this.
It's like, I don't like that there's this like layering violation happening, but the implementation complexity to resolve this half committed state issue is a far more serious issue than the, I don't like the layering violation.

Speaker 1: 00:34:53

Interesting.
Okay, cool.
Yeah, I think I got this.
At least I wrote it down.
Okay, yeah, I also wrote down sort of like the bulletins you gave me there, and I also got the transcript as well, But yeah, interesting.

Speaker 3: 00:35:03

Yeah, I think the message, the commentary back and forth on the issue is more than complete in terms of explaining why we made the choices.

Speaker 1: 00:35:14

Gotcha, I mean, okay.
So basically it's the job of like a protocol waiting to know when things are done because we reconnect basically.

Speaker 3: 00:35:23

Yeah.
So this is the spec change I still need to author.
I look at the draft to the quiescence PR.
But what it essentially means is that only one downstream protocol, like quiescence essentially creates this like session token and that session token is given in like an ownership sense to the downstream protocol.
So we can't actually batch these multiple fundamental upgrades into a single quiescent session as a consequence because of this implicit termination.
So what we do is we bind any sort of terminal state of the downstream protocol state machine to the terminal state of quiescence.
So they both terminated at the same time.

Speaker 2: 00:36:05

But as

Speaker 3: 00:36:05

a result, it means that you have to reacquire quiescence.
So if you wanted to do quiescence and splicing, then that terminates the whole session.
And then you need to do quiescence and then dynamic commitments if you wanted to do both.
You couldn't do quiescence and then splicing dynamic commitments and then unquiesce.
Like, that's not a valid protocol flow anymore.

Speaker 0: 00:36:24

And I think it's actually a good thing that you have to reapply, because usually, while you're quiescent, you will have comments that are being queued up to fail or fulfill HTLCs, and that you need to wait for the questions to end.
So it's really a good thing that even if you end it and the other side has nothing to apply and sends the TFU immediately, you can still get all your updates done before you also send STFU and then get another session.
So that gives you an opportunity to make sure that everything that is happening for, that is waiting for settlement, can be settled before the next fundamental thing.

Speaker 1: 00:37:00

Gotcha.
OK,

Speaker 3: 00:37:01

yeah.
I think we all talked about it.
Do you have a dialogue?

Speaker 1: 00:37:03

I guess just moving from implicit to explicit.
Or sorry, explicit to implicit.

Speaker 4: 00:37:08

But that's

Speaker 2: 00:37:08

your how

Speaker 3: 00:37:08

it is.
If you think about the UX issues as well, it's like if you're on RPC being like, hey, I want to do a splice, right?
That's like going to take by the time the splice operation is done and the user gets the feedback like you don't still need to be in quiescence that thing can resolve it's like resume it's like internal like function and like, you know, clear out various cues and then you might be like, oh, even if I as a user want to do these things back to back, there's going to be plenty of CPU time in between them to keep things in sync.

Speaker 5: 00:37:41

It's

Speaker 3: 00:37:41

like, I can't imagine a time where I'm going to want to batch the operations together, especially since I don't think that there's really any benefit to batching those multiple fundamental things.

Speaker 1: 00:37:52

Very cool.
All right, I'll check out the thing.
And I guess if you have anything that needs to be changed from that.

Speaker 0: 00:37:59

OK, And I'll reach out to you, Kegan, to resume the cross-compat test that you had been starting a few weeks or months ago.
All right.
So related is the splicing PR.
I finally opened the concurrent splicing PR because the existing splicing PR was really old, was really hard to read because there were a lot of things that were still there from the very early draft of spacing and didn't look like any more to how spacing works today, which was really confusing to read and really hard to fix.
So instead, I just restarted from scratch and rewrote it to also match what we currently have and that we've seen working.
So I would recommend reading that PR instead of the initial one.
Also because I added a lot of test vectors, why detailed protocol flows, and I think it makes it really easy to see the edge cases, the handling edge cases that can happen with splicing and make sure that your implementation does correctly implement nasty disconnection or nasty concurrency with splice.message.
So whoever wants to review splicing right now should take a look at this read.
And CLN is looking at it right now to make sure that they potentially update their code base to match.
Cool.
Yeah, I

Speaker 1: 00:39:18

mean, I think we definitely been waiting for a fresh one, because the other one was like comments of to-dos on to-dos, basically.

Speaker 0: 00:39:28

Yeah, and there was a comment by Keegan to make it an extension bolt.
I don't have a very strong opinion on that.
Most of it is already in its own subsection, so it could be moved to an extension bolt as well.
The only thing that I think is important is to keep the TLV definition next to the messages where they're defined.
For example, we define TLVs for commit SIG, TX add input, and TX signatures.
I think that has to be defined right next to the definition of this messages to otherwise, you don't want to be jumping from one place to be able to see all the TLVs defined for a particular message.
But apart from that, moving the core part of splicing to an extension bolt could work as well.

Speaker 1: 00:40:08

Yeah.
And also, interactive TX is already merged in, right?
So, everything on the risky rebase, okay, it goes to the control.

Speaker 0: 00:40:15

Yeah.
That's also why I wanted to start fresh with the latest state of master because it's easier to reference everything.
The only thing that is still dependency and is not much is Questions, but we actually only need one link to it.
So that is just a dead link right now in my PR, but once Questions is on master, this won't be a dead link anymore.
All right, so whenever people have time, they can just have a look at this spec PR.
And it's the same for liquidity ads.
I also created a new one that is more flexible and allows people, mostly BLEEPS, for example, or extensions to define new ways of paying for liquidity.
And the goal, I'm trying to set up a bleep specification for under-flight funding, like LSP under-flight funding stuff that would use this new format for liquidity ads as a proof of concept that this works.
So I think I should be done with that in a few weeks and maybe that will help review that liquidity ads proposal.

Speaker 1: 00:41:23

Sorry, you said that you're working on a POC of what?

Speaker 0: 00:41:26

On the type lending, basically the LSP stuff, where if you what happens if you are if you want to relay an HLC to one of your mobile clients they don't have enough balance.
How exactly do you do negotiate on the fly liquidity and I want that to rely on liquidity ads and splicing as much as possible because it really makes it much better in terms of trust in most cases.
And now that we have splicing almost finalized and liquidity ads potentially making progress, I think it's a good idea to try to specify that.
And

Speaker 1: 00:42:03

how's the liquidity ads come into play though?
I mean, assuming like the node that we're hiring is going to be one that opens, right?
Or are you trying to like disintermediate that?
But then wouldn't that require like pre-image transport across those nodes basically?
So like an interceptor type of thing?

Speaker 0: 00:42:19

No, what I mean is just a way to give your rates to your mobile client so that on the fly you are easily, the mobile client is able to pay for additional liquidity and use zero-conf to add liquidity on the fly and receive HTLCs that they would otherwise not be able to receive.

Speaker 1: 00:42:41

Gotcha.
But I guess, isn't that like dependent on the HTLC because it's sort of like an extra routing fee, right?
And that like, you know, if they're getting one BTC HLC, they get like 0.9, right?
No, no,

Speaker 0: 00:42:54

no.
The goal of the latest liquidity ads proposal I'm making is that there's flexibility in how you pay it.
So for example, you could, if you already have a channel and you would need to splice new liquidity to add the new HCLC, maybe the mobile wallet has some balance in that channel so they could pay the fee from the balance in that channel and no one else, the sender, doesn't need to know about it.
And even if they don't, then it could be shaved of the value of the HLC when it is relayed so that the sender of the HLC doesn't have to know anything.
My goal really is that the sender doesn't have to know anything and it only happens between the LSP and the mobile wallet and no one else has to know about the fact that the funding is happening.

Speaker 1: 00:43:38

Gotcha.
Well, it just seemed like this was different from liquidity ads.
I feel like this is like negotiation of like a channel, right?
Which I guess can't use that for advertising, but you need to get messages somewhere.

Speaker 0: 00:43:46

What Liquidity Ads does, it is actually just you can actually do it with just reusing Liquidity Ads.
So that's why I want to avoid adding something else if we can just reuse tools that we use everywhere.
And Liquidity Ads is a great tool here to just advertise how much it would cost to add liquidity on the fly and just reuse the exact same flow.
And right now we just use the plain liquidity ads and splicing spec in Phoenix and that worked great.

Speaker 1: 00:44:13

But one last question, I guess you're assuming that the user is always going to accept, right?
Because if they, I mean, otherwise they need something to reject.
I guess, how would they reject in theory?
Is this a time-based thing?

Speaker 0: 00:44:23

Or?
OK, so the way we do it in Phoenix right now is that users set a fee budget.
They set a static thing saying my fee budget is that much.
And if it fits the fee budget, then it's automatically accepted by the wallet.
If it doesn't, it's rejected.
And but otherwise you could also imagine sending a notification to a user and waiting a small delay.

Speaker 1: 00:44:44

Well, I guess that's what I meant.
Like that, if you were trying to do it all in protocol, you would need some rejecting you.
But I guess you're saying you're already contacting them, the notification or something like that.

Speaker 0: 00:44:55

Yeah, and it is possible to reject it.
It's just a matter of how do you decide to implement your wallet.
And you can always reject it, both on the RSP side and then the wallet side.

Speaker 1: 00:45:06

Gotcha.
But I mean, but then wouldn't you still need something else to like, to ensure that the users, like, you know, wallet basically accepts the smaller value invoice payment?
Or you're saying that like, this is implicit.
You don't necessarily need anything for that.
And they know because I accepted the liquidity ad, like I'm going to accept the 0.9 BTC HCLC?

Speaker 0: 00:45:25

Exactly.
That's going to be part of the liquidity ads format to say, I'm going to fund that much on that transaction.
And in exchange, I will take that much in fees from the HLCs that I will relay next.
So that's part of the thing that we were defining a way to communicate that.

Speaker 1: 00:45:42

Okay.
Yeah.
I think I actually need to check out the latest PR to understand.

Speaker 0: 00:45:47

I think the easiest way would be to wait for the blip that I will be writing about that, that should contain all the information and link to the liquidity ads and splicing parts that make sense for this.
And I think this should make it clearer.

Speaker 1: 00:46:03

Cool.
And then so 11.53 is the one, because I see there's two liquidity ads.
Right.
So I see 11.45 and 11.53.
One just says advertise, the other one is extensible.

Speaker 0: 00:46:14

Yeah.
So basically, 11.45 was just mostly 8.78 without the enforcement of scripts, but with only one format of liquidity ads, and it is actually limiting and cannot easily be used for more LSP spec, more LSP stuff.
So that's why I then created 11.53 and I explained in the description that this is a more extensible format that allows more ways of paying the fees to be added and all used in extension.

Speaker 5: 00:46:48

Gotcha.

Speaker 0: 00:46:52

And Lisa is supposed to be reviewing it, but she gave me a concept on the latest one, that more extensibility is probably a good idea because the initial format that she was using was really very tailored for the specific case where she wanted to add an addition to the script to make sure that it was a CLTV locked.
But if we're not going to do that in some cases, then it makes sense to have more flexibility and define more ways of selling liquidity.

Speaker 1: 00:47:24

Cool.
Okay.
All right.
One last question here.
So, you know, whenever, like, or maybe this is like going to be in the blip, you know, perhaps like, you know, whenever you're sending an open channel request, is it going to include the payment hash of the HTLC you got?

Speaker 0: 00:47:37

I see

Speaker 1: 00:47:38

it, yes.

Speaker 0: 00:47:39

It could.
Actually, it's really the other way around.
The way I'm planning on doing it is that whenever you cannot relay, as VLSP, you cannot relay on HTLC, you're going to send a new message that will add HTLC that contains basically all of the HTLC but isn't tied to a channel so that the mobile wallet receives all the earnings and verify that by decrypting the onions, they receive a whole payment and it's a payment they would accept if they had a channel.
And then the mobile client is initiating the open because they know that they are expecting this HTLC to then be relayed on the new channel.

Speaker 1: 00:48:14

Gotcha.
OK.
And then I guess while we're here, I mean, do people think it's useful to have that new share, have the HTLC from the start?
Because otherwise, I guess, aren't you in, well, I guess it depends how you pay for it.
Because otherwise, you have the thing where the user, the server basically opens the channel, but you don't do anything, right?
Versus like having, they can reject the HTLC, just having it there.

Speaker 2: 00:48:35

Yeah, you always have that.
So I think the distinct, so the reason why you have the will add HTLC message is less to fix that issue and more just to handle MPP because you need to handle MPP somehow, you need to know like, hey, is this, you know, I have three HTLCs, is this a full payment or are you still waiting for more?
The way you address the like, and probing, right?
The way you address LSP trust client versus client trust LSP is just a question of when you broadcast the transaction for a zero-conf channel.
So you would do In either design, you open the channel, you do your normal channel open flow, and you send the client, the LSP sends the client the update at HTLC messages, you do the whole dance, and then the only difference is either the client immediately responds with the payment pre-image, whether the LSP has broadcasted the funding transaction or not, or the LSP waits to broadcast the funding transaction until after they see the pre-image.
So either way, you have the exact same message flow.
It's just a question of when that transaction gets broadcast and whether the client checks the mempool or something.

Speaker 1: 00:49:51

Yeah, but I guess my question is, do people think it's useful, if we're doing the channel thing in a separate format, to add a push amount HTLC?
Or you're not worried about that gap?
Because this would basically have the first date have the HTLC,

Speaker 2: 00:50:04

is what I'm getting at.
Right, but that's something I think

Speaker 1: 00:50:07

we discussed before.

Speaker 2: 00:50:07

My point is that that gap's not a gap.
My point is that gap is not a gap if the LSP waits to broadcast the funding transaction until after they see the pre-image.
You're just adding more messages and more additional flow that's not actually required.
Because the real thing is the LSP is just going to wait to broadcast the funding transaction.

Speaker 1: 00:50:26

But isn't it risky for the user to send before because then the LSP can just not broadcast, right?
And just take the HTLC.

Speaker 2: 00:50:33

Yeah, so you can't fix this problem.
There is no way to fix a zero-conf JIT channel, or any kind of JIT channel.
Always, either the user can ignore the HTLC, not provide the pre-image, and grief the LSP and waste their liquidity, or the LSP can steal the money because the user sends the pre-image before the funding transaction is actually out there.
There's no way to fix that.
You just have to pick one.
And so I think the intention, and I don't want to speak for Bastien, but I think the intention is that the spec will support both.
But the spec will support both purely just by saying the LSP can broadcast the funding transaction immediately, or they can wait.
That's up to them.

Speaker 1: 00:51:21

OK.
Yeah, I guess it's like zero comp spend, basically.

Speaker 3: 00:51:30

And they can

Speaker 1: 00:51:31

double spend it anyway.
Because even if you added it, they can still double spend basically is what you're saying.

Speaker 2: 00:51:35

Right.
You would have to wait for one confirmation.
So yeah, the client would have to wait for one confirmation or the LSP has to wait to see the pre-image to broadcast.
But you got to do one or the other.

Speaker 1: 00:51:47

It comes.

Speaker 0: 00:51:52

But one of the things that I will detail in the blip is that this really happens when you create the channel initially.
But then when you're splicing, most of the time, the liquidity fee can be directly paid from the user's channel balance.
So then you don't have this issue, because you make them pay the fee for the liquidity they're buying, and then you relay the HCLCs. And even if they fail it instead of fulfilling them, they have already paid a fee for the liquidity.
So it's OK, and they cannot cheat as much as for a new channel.
So it makes it better and shifts the issue to the initial channel creation.
And cases where the user doesn't have any balance in the channel, but it's not in every on the fly funding attempt.
But I will detail in the blip because it's hard to grasp, and it makes a lot more sense when we detail every scenario.
Cool.

Speaker 1: 00:52:47

Yeah, looking forward to that.
That was something I think is needed, just to fill the gap there, and we'll use all the same thing there.

Speaker 0: 00:52:54

Yep.
All right, so we are already at one hour.
Is there another topic that someone really wanted to discuss today?

Speaker 1: 00:53:05

Yes, I know Rust isn't here, but I don't know if people are up on gossip V2 stuff, basically.
So I think, you know, I need to check on the place where I think Elle was able to be here, where she asked me to like pose two questions, basically, primarily around like, the motivation of like, re-announcing, you know, old and new channels using the protocol basically.
Because it feels like if we can cut that out, then you don't have to worry about questions like, how do you acquiesce to the timestamp versus the block height?
And then also, do you continue to retransmit both messages?
Do you continue to risk you retransmit both messages?

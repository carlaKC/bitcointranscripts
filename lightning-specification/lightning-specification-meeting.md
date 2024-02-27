---
title: "Lightning Specification Meeting"
transcript_by: carlaKC via tstbtc v1.0.0 --needs-review
tags: ['lightning']
speakers: []
categories: []
date: 2024-01-29
---
Speaker 0: 00:00:05

Okay, so it's like T-Best said he won't be able to attend, but he's nicely created an agenda for us, like usual.
So I can take a break and run him through the list.
So I guess first up, so I think this is sort of like went back and forth a few times.
This is like the very long-lived request to just have a zero value in reserve basically like a first class type.
T-Best made this PR, so it's checker 1140.
I think there's back and forth thing we realize that you also need to signal that you actually accept it, not just that you'll send it, just to make sure that things are just work without having some manual intervention.
Pretty straightforward.
We were wary of getting into a state inadvertently where a channel gets stuck because of some weird fee situation.
So one thing we're doing now in 0.18, something I think that was a very long and old portion, basically, I think that buffer thing, we didn't have the fee buffer.
So basically, at times, we would have issues.
Also, now we do a thing now where we have some first compatibility with C-Lightning where we would update the fee too quickly.
Basically, we have to update our in-specs as well.
But that's the only thing I think of implementation-wise as far as zero reserve.
I know people do it in the wild, but that doesn't mean they aren't hitting a fee-related XP.

Speaker 1: 00:01:23

Yeah, so I think the spec says, you know, you should fee reserve, you should be able to handle like a two times your current fee.
And we actually wound back that.
I don't know if we actually merged that PR for this release or next release, but if your fees are also historically really high, we figured the chance of it going to twice that is less.
So we have like a sliding scale where we'll kind of have a fee buffer of like down to 10%.
But I can't remember exact numbers.
Christian had some numbers.
So the idea that you should basically ensure that you can still get something through, even if the fees were to jump by some factor or more, we've kind of dialed that back a little bit.
But yeah, basically, there's nothing objectionable on this feature that I can see, right?
I mean, don't use it if you don't want it.
And if you want it, now it'll be there.

Speaker 0: 00:02:18

So. Yeah, yeah, yeah, same.
Something that has been around for a while until an official patch, I guess like we should bring it in.
It's just like happened with zero conf.
Yeah.
Period of time, so.

Speaker 1: 00:02:27

Yep.
So yeah, just yeah, someone else has to implement it and then do interop testing.
And I mean, you know, that should be pretty easy.

Speaker 0: 00:02:35

Yeah, yeah.
I mean, yeah.
So I think we also need to make a tracking issue for it as well.
Right, and I'm sure I can just poke someone.
I know, I think Breeze has probably a few different versions of this patch sitting around, maybe they can just contribute upstream assuming it applies cleanly.
It's not some of the most important fork.
Yeah.
Cool.
I'm guessing Claire has one already.
Because I think they've also had this deployed for some time just between their nodes and their mobile as well.
So I guess we should catch up on the interop side with them.
Cool.
Check.
All right, Trampoline, seems like trampoline's back.
I think LDK and Eclair did a bunch of work on it.
I think just the packet format was the last thing I remember.
I'm not sure if Rx here or LDK.

Speaker 2: 00:03:32

I'm here.
Yeah, LDK.
I was sick last week and I was making slower progress, but I have implemented unit tests and have full test vector parity with what Dbast has produced, which I'm really excited about.
We have the ability to be constructing trampoline packets, so now I'm just building wrappers around that all the way until users are going to be able to just call methods.
So the next level that I'm currently working on is constructing a Trampoline packet based on user specifications, like the structs that users are passing.
And the next level of abstraction will be pathfinding that will incorporate trampoline-rod construction.

Speaker 3: 00:04:29

Cool.

Speaker 1: 00:04:30

You have to make some tiny modification to the spec so that you can name it the 2024 edition because it hasn't been updated in three years.
It's currently on 2021 edition which I know was a joke at the time but it's now, you know, kind of old.
So yeah,

Speaker 3: 00:04:45

if you could come

Speaker 1: 00:04:45

up with some trivial modification, that'd be great.

Speaker 0: 00:04:48

Were there any changes to the format that happened or was it just sort of like just coalescing towards what they already have?
Is there

Speaker 2: 00:04:54

a moment- No, no, no.
I briefly suggested some changes that turns out would have made it more like Rendezvous routing.
The changes were about not nesting onions and just keeping everything at the same level, unwrapping and rewrapping, and then unwrapping again.
But it wasn't worth the additional complexities.
So we're just keeping it as it was originally proposed.

Speaker 0: 00:05:17

Got you.

Speaker 2: 00:05:18

Now, I guess the only changes are really the TLB IDs, which Bastien can expand on if he's here.

Speaker 0: 00:05:26

Cool.
Then the thing about the PR, so I guess the one with 836 is like the version, Or there's two PRs, right?

Speaker 2: 00:05:38

I keep having to re-Google and reopen them.

Speaker 0: 00:05:42

Yeah, so one is 8.36 and the other one is 8.29.
8.29 is the 2021 edition, that's what I see you saying.
Yeah.
I didn't cop them.

Speaker 2: 00:05:51

There is one that most recently has been modified by tbast to incorporate updated test vectors.
That is the definitive one.

Speaker 0: 00:05:59

Yeah.
It looks like this one has been updated most recently.

Speaker 2: 00:06:02

Yeah, Q6.
Then that's correct.

Speaker 0: 00:06:04

Okay, cool.
Okay, I'm just curious.
Cool.
Cool, and then, no, we've been just catching up a bit with that internally just after as we're working on blinded path stuff like I'm starting pathfinding so it seems like you know they're related and can be composed that's cool but not like next release or anything like that we're just trying to make sure we can get stuff into this one the next release should be like March or at least like RC there's RC in March and then we'll go from there.

Speaker 2: 00:06:33

Yeah, I think with LDK, the question really is how much functionality do we want to incorporate?
Because if it's just trampoline serialization, we already have it.
If it is trampoline package construction, PR is open.
We should be able to have it too.
But like-

Speaker 0: 00:06:53

Have people thought about like invoice stuff and how that combined with, you know, both 11, both 12 offers up like that.
Like, you know, is it, I guess it's smaller than blinded paths or I guess, well, it's fixed size, right?

Speaker 3: 00:07:05

I don't think you, I don't think for trampoline you're ever gonna include it in an invoice.
It's more a thing to send to a known node.
Yeah,

Speaker 1: 00:07:17

yeah, The client goes, oh, I have no idea how to route this, so I'm going to bounce it through the trampoline kind of deal.
So I guess you don't need a...

Speaker 0: 00:07:24

Sure.
That makes sense.
Okay.
Yeah.
This is like just packing construction because they don't have the...
Okay.
Yeah, yeah.
Sure, sure.

Speaker 1: 00:07:30

Yeah.
Well, what do you need?
There's a complimentary technology, which is, there is a server-side idea, which is kind of similar, where you pay some node, HandWave, to route stuff for you, and then you just redirect all your stuff to that node, and it figures out how to get it to you.
But that's kind of a separate, that's the other half, the incoming side equivalent of this, which doesn't exist yet.

Speaker 0: 00:07:53

Richie, something on it?

Speaker 2: 00:07:54

Yeah, I was just thinking about whether it might make sense to incorporate in the invoice whether or not a recipient supports Trampoline.
I don't really think it necessarily makes sense because it is not the Trampoline node that receives the money, the Trampoline node is just the penultimate hop.
However, would there ever be a scenario that we would want support where we are sending money directly to a Trampoline node?
I don't...

Speaker 3: 00:08:16

Wasn't there a case...
Eclair had some thing where they were...
I think like zero amounts were...
The trampoline could steal some of the, because like the, I mean it didn't commit to the amount.
No, that's not right.

Speaker 0: 00:08:34

There was some...
Was that pre-payment secret?

Speaker 3: 00:08:38

No, it was post-payment secret.
They had some...
Something where Trampoline still wasn't safe, but I don't remember what it is now.
So maybe just ignore it and maybe it was fixed and we'll pretend it was fixed and ask faster in two weeks.

Speaker 0: 00:08:55

Okay.
Yeah, I don't really remember that.
Okay, cool.
All right.

Speaker 1: 00:09:03

Yeah, roast beef, hide that in the minutes somewhere.
See if anyone reads them.
There you go.

Speaker 0: 00:09:06

Yeah, yeah.
Question mark.
Like, is there active vulnerability?
No. Okay, cool.
Next thing.
Template.
Okay.
I think it's like an offers thing around like another way to specify a node ID.
Just so it's like less bytes.
This is 1138.

Speaker 1: 00:09:29

Yeah, waiting on me for interop, I think.

Speaker 0: 00:09:34

Oh, it's like a multi-format.

Speaker 3: 00:09:36

Pretty straightforward.

Speaker 1: 00:09:36

Yeah, it's like we've got 02, 03, why don't we just use 01 or something for, you know.

Speaker 0: 00:09:42

I see.

Speaker 1: 00:09:44

And there's possibly a few places

Speaker 4: 00:09:45

we could use it, but

Speaker 1: 00:09:45

the most obvious place is we're in offers where people basically want to have a route and they want to say, go via this, because that's the ones that tend to be in the QR code, that's in people's faces, and it's like, length really matters.
It's not particularly useful if it's a long live QR code that you wanted to tattoo on your ass, but it's because you're counting on that short channel IDs being around forever.
But for many things, it would be perfectly acceptable.
So kind of makes sense.
And I think it's the least objectionable of all the different ways to encode it.
So yeah, everyone hates it, but it is smaller, right?
So, yeah.
So waiting on interop, I think.

Speaker 0: 00:10:36

Okay.
Cool.
I've

Speaker 5: 00:10:41

got a somewhat related blinded paths question for everyone while we're here.
Sure.
I just wanted to ask, what is everyone currently doing for Mac CLTD expiry for a blinded path?
Because I was testing my LND stuff the other day and I ran into a zero value.
I think it was on like an old version of CLM that I could get to build on my computer.
So I just wanted to check, like, the zero value doesn't really make sense there because it's the highest height, which is except the payment.
Does anyone know offhand if they are setting that to zero?

Speaker 1: 00:11:15

Not deliberately.

Speaker 5: 00:11:18

Okay, so I shouldn't, yeah, cause right now I was like checking if this is a zero value, just pretend it's not there because it's in like a composite field.
But that sounds like the wrong thing to do.

Speaker 1: 00:11:30

That does sound like a bug.
Yeah.

Speaker 5: 00:11:33

Okay.
I'll see if I can reproduce that and then check it out.

Speaker 1: 00:11:37

Yeah.
Ping me about it because that's wrong.

Speaker 5: 00:11:40

Okay.
Great.

Speaker 0: 00:11:46

Cool.
All right.
Check that one.
Channel jamming.
So I guess there's this state thing.
But I guess any general updates or anything here?

Speaker 5: 00:12:00

No, no updates for me.
I've been kind of trying to get the route planning stuff done in L&D.

Speaker 0: 00:12:05

Gotcha.

Speaker 5: 00:12:06

I think this week.
Cool.

Speaker 0: 00:12:08

Yeah, and we have someone new starting, I guess, tomorrow.
And I think one of the other things we'll be starting to take a look at, like the blip and stuff like that, just in terms of like, you know, integrating more deeply into on the or just having someone that is up to date on all the ways and goodness there.

Speaker 5: 00:12:22

Nice.
You did mention Lalu last, last spec meeting, you mentioned there was something else that you might want to be passing along with update ad.
Do you remember what that is?

Speaker 0: 00:12:34

Yeah, it's for the tap stuff.
It's just like an asset ID value for the channel thing we're working on.
And it matches that, hey, where do you need to depth plumbing?
So just so that we make sure that's available.
I haven't started to dig into it yet, just other stuff.
But hopefully this week I'll be able to take another deep look at it.

Speaker 5: 00:12:51

Okay, sweet.
I just get an idea of the APIs that that would need.
But I'll ping you on the on Andy's Slack.

Speaker 0: 00:12:59

Yeah, that's not a good thing.
Yeah, something that I'm figuring out.
So It's either just going to be directly on the API itself, maybe there'll be some other config thing.
Maybe it may also just be a top-level config thing, so not necessarily at the RPC level, but something that you can do if you're making an LME node in a cool process basically.
But so kind of like finish finally that's when designs that we should have to think about that.
Okay, right.
Cool.
Cool.
All right.
We got the newest, the newest thing on the block, DNS-based offers.
I guess we talked about a little bit last the other week.
Yeah,

Speaker 3: 00:13:35

I don't think there's very much to talk about here.
I need to follow up on the offer set of things.
I think the way forward that makes the most sense is to rip up the current design and replace it with just including the user and domain parts in the invoice request.
That's nice and easy.
But the next step there, I think if I recall correctly, was to define some kind of blip type range for the offers and invoice request and invoice so that we can define those in blips.

Speaker 1: 00:14:18

Yes, that's been sitting in the back of my head for quite a while, is that we don't have, so it defines well which parts you're supposed to mirror and you have an offer, you're supposed to copy the things in the invoice request and the invoice request copies things into the invoice.
You end up with everything in the invoice, but we define those in very strict ranges and we really should probably add an experimental range and define what happens if you don't understand something.
Do you copy it or not?
Do you rely on features to say which bits you understand and you just blindly copy all the fields, even if you didn't understand them, or is understanding them like an acquiescence that you accept the terms of whatever it is that you just copied?
Someone needs to think kind of hard about that.
Somebody else.
For our coffee.
Well, if it's even, then you won't do anything with like, because if you see an unknown even field, you're like, I don't know what this is, you're out.
So that already has a semantic, right?
So you will be an odd field.

Speaker 3: 00:15:18

That's magic.
You want like the person who include whatever who creates the thing can decide whether they want it to be either you understand this and you agree with it or you fail or you ignore this if you don't understand it.

Speaker 1: 00:15:34

Right, yeah, but it's the latter one.
You're like, so are they not gonna copy it because they didn't understand what it was?

Speaker 3: 00:15:41

Oh, they need to copy everything.
Otherwise the

Speaker 0: 00:15:45

data we put

Speaker 3: 00:15:45

in the metadata might not make sense.

Speaker 1: 00:15:47

You're hashing scheme breaks.
Yes.
So I think you want us to copy everything.
In which case, you go, how am I going to tell whether or not you got it?
If I'm supposed to say, I'm trying to handle both, right?
I've got some optional field.
And if you understand it, great.
But if you don't, I can fall back or whatever.
Then you kind of need to know now maybe we need some features in there and you will, you know, if you understand, you know, feature 3015, then you know, you understand that field, you will set the feature bit and then you'll do it.
The point is it's gotta be defined.

Speaker 3: 00:16:21

I don't see any hard metal ways of copying.
Like if you don't understand it, oh well, but you should always copy because the hashing scheme for us, sure, yes, but also there's just no reason not to.
You might as well.
There are certainly schemes where you want it, and then there are schemes where you don't, I don't know.

Speaker 1: 00:16:44

Oh, well, we do require some mechanism to indicate whether or not you actually understood it.
Sure.
That can be separate from inclusion.

Speaker 3: 00:16:51

That can be separate.
Yeah.

Speaker 1: 00:16:53

Yeah.
Cool.
So yeah, somebody needs to sync think through all this, and this is an opportunity to do that.

Speaker 0: 00:16:59

Yeah.
Then just one question.
This is so a single node can respond to offers for like many different like user registered users basically, right?
And like, that's the mapping here.
Yeah.
For this particular.

Speaker 3: 00:17:10

Basically, the intent would be that you would display you'd have one offer that just goes to you, you know, big custodial service or whatever.
And then the invoice request says, hey, I actually want to pay this user specifically.
And then you could say no, or you could say like, great.
And then you like, you know, hide it in the blinded path so that when you actually receive the payment, hidden in your little encrypted blob, it says this is for user XYZ, then you can credit it, and it's all nice and stateless, and seamless, and yada yada.
Cool.

Speaker 0: 00:17:46

Yeah, you got to put the trio of docs.
Next, support mutual close.
I was close last week, I'm even closer this week.
Just doing some final unit test basically.
Just some of these edge cases.
I should be able to just get the PR out of draft this week, and then start to hit up T-BAS or whoever's around on Async side for some interop testing basically.
But yeah, the only things that I need to just make sure I'm aware of, as case-wise, some of the signature combinations basically, because I think I've implemented two-thirds, the one that made sense, but I should just do the other ones and you know, when it's like something complains by sending it.
Otherwise, wasn't too bad.
I just, you know, it was an opportunity for me just to like do things in a very new way and L and D, which is why it took long.
I made some stuff on the side, refactors, et cetera.
But otherwise, like I'm excited just to like get people caught close and actually works.

Speaker 1: 00:18:35

Yeah.
Cool.
Yeah, I haven't gone back and revisited the spec because I have not even implemented it.
So I look forward to you having ironed out all the stupid ideas from it and then I can just follow and implement it.
Cool.

Speaker 3: 00:18:45

Implement it.
Cool.

Speaker 0: 00:18:49

All right.
Okay.
Questions?
I think on this, T-Bass and Keegan are looking to do some interop testing between L&D and Declare.
There's a scenario y'all are discussing, I think, Keegan?
We're going

Speaker 6: 00:19:12

through how we want to go about testing, because some of the stuff is like, it's like we're in a lot, we want to mess around with the internals or do we want to just see outside users and I currently through a like working on like an explanation for why we shouldn't actually try to put a whole bunch of internals instrumentation to do this interop testing.

Speaker 0: 00:19:43

And by that do you mean something like some debug RPC to just send STFU or something or?

Speaker 6: 00:19:49

Well, or like sort of measuring internal state about whether or not, like which one thinks it's the initiator.
I don't think it's worth it necessarily, but.

Speaker 0: 00:20:02

Interesting.

Speaker 6: 00:20:03

I'm gonna be like writing all this up and putting it in.

Speaker 0: 00:20:06

Okay.
But I guess TLDR is that you feel like it's just better to test it in concert with something like splicing that actually uses it for an end goal versus saying, am I flushed or something?

Speaker 6: 00:20:15

Yeah.
I'm trying to say we should be testing for things that are externally observable rather than trying to like query internal state because you know it's just I wouldn't recommend like creating instrumentation in the internals of these implementations just to do interop testing.
That's the domain of unit tests, in my opinion.

Speaker 1: 00:20:43

Yeah, it's really hard to, because by itself, it doesn't do anything, almost by definition.
It just stops doing things.
Our tests are horrible.
We have this dev thing that can say hey send this and it will start doing that and then we look in the logs to check that both sides have done it and that there's no then that no activity had occurred has occurred otherwise right which is pretty pretty hacky yeah yeah you really can't test it until you have something on top.

Speaker 6: 00:21:10

What I would love to be able to do is to get a framework that does protocol traces, where we can then start to do like invariance checking over like each side of the, like basically get a message stream one way, a message stream the other way, and then just do like a whole bunch of checks about like what message orderings are allowed and stuff like that.

Speaker 0: 00:21:31

We kind of have maybe two or three versions of that in various levels of maintenance.

Speaker 1: 00:21:38

LN prototest was supposed to be that, but we've kind of abandoned it and we really need a good couple of rewrites and generalization and all those things.
But yeah, I mean, the important thing is that SDFU goes away when you reconnect.
That's kind of important.
And you reset so you don't get stuck.
And yeah, that you actually do shut up after you've exchanged SDFUs. And There are a couple of corner cases if stuff's in flight, but that's about it.

Speaker 0: 00:22:08

Well, because I remember this came up, like one question with that and splice, but let's say you have an actual channel, you do SDFU, then you do a splice.
Is the intent that you reconnect once confirmed or reconnect after the thing or is there something in the splicing that says, okay, we're no longer in the, you know, the STFU state?
Okay.

Speaker 1: 00:22:24

Because we're all- Because all

Speaker 0: 00:22:25

the STFU terminates, yeah.
Okay.

Speaker 6: 00:22:29

Yeah, That's something I'm not a huge fan of.
I wish I would have understood that earlier.
I thought that the point was that we were gonna reconnect after whatever thing took advantage of the stop traffic.
If we wanna resume, then we should probably include that in the SCFU proposal and not pepper it around all of the dependent proposals.

Speaker 4: 00:22:49

Dusty.
Where currently is when you receive the signature for the splice, then the SDFU is presumed to be ended.

Speaker 6: 00:22:57

Right, but like what that does is it deeply entangles the two proposals in a way that need, like just isn't necessary.

Speaker 1: 00:23:05

Yeah, and if we're going to do other changes, then you're like, well, should we have a STFU is done?

Speaker 6: 00:23:12

Yeah, and like, because like dynamic commitments is going to try to make use of the STFU as well.
And it's like, well, in that case, can we do a resume in the STFU proposal rather than having to now say, OK, in dynamic commitments, this is when we're no longer STFU.
And then this, like, it's like.

Speaker 1: 00:23:30

What if you wanted to do both?

Speaker 6: 00:23:31

Maybe we save a message, but do we really need to save a message?

Speaker 1: 00:23:34

Yeah.
So the reason it didn't exist is because the original STFU was for channel upgrade, where you reconnect to do it.
But forcing reconnection in other cases is just weird and disruptive, right?
Because you've got other channels, right?
Why are you reconnecting for this one kind of thing?
So yeah, I definitely don't want to do reconnect as the canonical way of resetting SDFU, although it should reset.
So yeah, I'd be happy with an SDFU finished kind of, you know, come up with a cool acronym that's the opposite of SDFU.
Yeah, that's right, yeah.

Speaker 0: 00:24:08

You could just call it OK or something like, SDFU, OK.

Speaker 1: 00:24:13

Whatever, whatever, dude.

Speaker 0: 00:24:15

Yeah, yeah, whatever.
No, yeah, that that that makes sense.
It seems like it would be like, you know, I guess not a very like elaborate cut out either and just like a way to decouple it like you're going to say as well.
So like you're not necessarily reliant on implicit sort of behavior in another protocol.

Speaker 6: 00:24:31

I mean, it's much easier to follow.
Yeah, yeah,

Speaker 0: 00:24:36

Then there's a clear termination state or going back to normal basically.

Speaker 3: 00:24:40

I think

Speaker 4: 00:24:40

there's some weirdness, right?
Like what if there are certain cases where the splice like can't be abandoned?
Like if I've sent you a signature I haven't gotten from you, right?
And then if you end the STFU in that point, that's going to be like, you know, obviously the default, the before close, right?
But it's like, it's not sure how clean it is to have, to have just say this STFU ends, couldn't always work.

Speaker 2: 00:24:59

We'll still be dependent

Speaker 6: 00:25:00

on that.
All protocols need a way to handle their state being essentially truncated, right?
So it's like if you abandoned your interactive TX halfway through and you started going with your splice, what happened, the downstream messaging.

Speaker 1: 00:25:16

It's equivalent to a disconnect.
Yeah.
If you receive it at a time you can't handle it, you hang up on them and let them try again.
Right.
And you're like, I'm gonna pretend I didn't see that.

Speaker 4: 00:25:27

This is implications for like channel reestablished.
Cause there's certain cases where I've given you certain signatures and I haven't gotten them back and they get really complex to handle with re-establish and I can't think through like live right now what they would mean with an S if someone's doesn't at end ST if you command at one of those moments like what is the correct thing to

Speaker 6: 00:25:47

do at the same time like I think

Speaker 4: 00:25:49

those are well defined

Speaker 1: 00:25:51

I think you treat them the same.

Speaker 0: 00:25:56

That's sort

Speaker 6: 00:25:56

of my point.
If you already have a mechanism to resolve this like halfway state commitment, and like just use it for any sort of situation where like you receive a message out of time and if you don't have one then you have bigger problems right like

Speaker 4: 00:26:10

yeah we can do it's just it's just particularly this is like the re-established stuff gets really complex really fast And I'm not saying it does make it more complex, but it's something that I would

Speaker 0: 00:26:18

think about.
So, Dusty, without something like this, what do you do in that case?
Let's say you're saying we're doing the splice, you only get one signature.
The channel is still also just stuck, right?
You can't do anything basically with that, right?

Speaker 4: 00:26:29

In certain cases, yes.
In certain cases, no.
What you do is you reestablish and ask for the signature and stuff that you're missing.
And you demand that to restart the channel.
Without that, then you

Speaker 0: 00:26:38

have to force close everything.

Speaker 1: 00:26:41

So yeah, if you receive

Speaker 6: 00:26:43

an unexpected

Speaker 1: 00:26:43

packet, you would hang up.
You'd warning out, and you'd hang up.
Or maybe you'd force close.
But probably you'd just hang up and hope they fix it next time.
Similarly, if they try to splice halfway through a splice and they send you other rubbish.

Speaker 6: 00:26:59

Or Send an update ad, right?

Speaker 1: 00:27:02

Yeah, that's right.

Speaker 6: 00:27:04

That's the thing, we can send you any one of a thousand different things that are going to fuck your whole world.
And it's like,

Speaker 4: 00:27:10

I don't think there's a way

Speaker 6: 00:27:11

to sidestep it.

Speaker 4: 00:27:13

99% of the splicing, we just drop the splice.
There's just these key moments when like the signatures are ones that has and ones that doesn't where it starts to get you have to be very careful with it.

Speaker 1: 00:27:24

Yeah so I think I think you would treat it as as if they've disconnected except you would disconnect you go you'd send a warning ideally saying I don't know what the hell this is.
You can't do that yet.
You would disconnect and hope it doesn't happen again.
And file a bug report.
So I don't think it would be too bad.
The only problem is that the breaking of existing implementations, if you don't expect the SDFU to be terminated, if the other side expects you to explicitly terminate the SDFU and our implementation doesn't, what happens here?
Because SDFU termination would probably be an even message.
So in theory, this would be a feed that change.
So we'll have to, you know, transient problems.
We'll sort out some way of detecting that.

Speaker 4: 00:28:13

Yeah, I think it could work thinking about it more.
But one of the weird things is when you go through the reestablished mode, there isn't currently an explicitly defined STFU entering, and yet the code will assume that you leave the STFU when the splice finishes reestablishing, which is something I've been thinking about maybe document more, but

Speaker 0: 00:28:31

I don't know.
It's complicated.
Well, yeah, because it seems like you could send it again.
It seems it's better to just have this explicitly be defined as well too, in either case, just because it seems like the state space, like you're saying, I agree with Rosalie, I'm really sorry.

Speaker 1: 00:28:42

In particular, the case where if, you know, in future we have a whole heap of things using SDFU for, in theory, you might want to do more than one at once.
And you can't do that at the moment.
You literally, they would have to be segmented, right?
You're like, you know, if we're going to change dust limits using SDFU or something, we wouldn't be able to do that at the same time as a splice because each one would terminate the STFU?

Speaker 0: 00:29:04

Ah, yes.
That's sort of the

Speaker 6: 00:29:05

point.
I think that you want the proposals to be modular.
Like anything that, the way I even see the splicing proposals, you have two different protocols.
You have like the main splice one, and then you like kind of go into a new protocol stack frame if you will when you go into interactive TX and it's like that but then once you've agreed on the TX then you kind of like you're discharging that protocol and then returning back to your main splice and the way I see it is like splicing and dynamic commitments sit on top of another protocol stack frame, which is your STFU.
It's like, I'd like to keep those stack frames properly.

Speaker 1: 00:29:45

Yeah, but on the other hand, you'd be like, from our point of view, you get a request to do a splice, or you get a request to do something from like the main daemon sends to you and you go, cool, I need to be in STFU.
And if you're already in STFU, I think We would optimize, you know, we're unlikely to force it in and out of STFU at that point.
We would actually just go, we're already in STFU, we will do this while we're here.
But yes, you need a separate loop that is your limited protocol for transaction construction.
I don't think allowing random shit in there is useful at all.
So I'm saying you can slightly blur them and you can buy an STFU.
I don't think you want to overlap them and have, oh, while we're in the middle of a splice interactive TX, I am also going to send you a request to change dust limit or something.
That is insanity.
So yeah, I share your frame.

Speaker 6: 00:30:36

And do something.
Doesn't also necessarily mean that we should.
That's the other thing.
Is that like, and if you guys want to experiment with like trying to like sort of bash them together and stuff like that, that would be fine.
But my point is that if we have a protocol that explicitly says, OK, this STFU stack frame is getting dropped, AKA now you may speak, or whatever sort of resumption there is, at least there's a clear view on both sides of the channel what the protocol state is.
Like regardless of how you want to handle the actual software that orchestrates it, it's like it's very cleanly separated and you can like look at the traces and see like okay this is the state that we're like in.

Speaker 1: 00:31:15

Yeah and I prefer more restrictive protocols like in interactive TX I don't want to have to handle anything other than interactive TX, right?
You should have like four messages or something that are possible, anything else is crap.
You should neither set nor receive.
That's way, way simpler than trying to handle the intersection, particularly intersection of messages that don't exist yet that we add to the protocol.
And then nobody thinks, oh, but how is this going to interact with?
Yeah, we don't want to go there.
I agree.

Speaker 0: 00:31:41

Cool.
Okay.
So it seems like we're looking in the direction of creating this if you terminate to just decouple it from other stuff.
Okay, cool.
Which leads us into our next topic, splicing.
I saw you like made some tool, Dusty, like sort of like, you know, like a DSL to communicate or like the sequencing of the splice stuff.
That looked pretty cool.

Speaker 4: 00:31:58

Yeah, we're gonna like, it's sort of like a, it's kind of scripting language, sort of.
I was like mocking up what complex spices would be.
And I was like, why don't we just parse this like a simpler form of doing it.
So, but core idea is just be able to take a spice of tons of channels and tons of deposit withdrawals and stuff and just merge it into one like one liner script kind of thing.
And I got really excited about it and went really ham with it.
And I think it's awesome.
But yeah, working on that.
I don't know if like that should be a spec thing.
But something I'm excited about.

Speaker 0: 00:32:30

Well, yeah, I see something useful to get a feel for API-wise.
I think an interesting thing as far as you're saying, sort of like a way you can declare it if you communicate batch splices across several different channels, which seems relevant for the big nodes out there.
Yeah, but I'm assuming maybe you'll get more experience with it basically, and we'll go from there because otherwise, API-wise, I haven't thought.
I guess I've only thought in the single splice, like single channel, single splice, obviously you can batch it on chain, and there should be a way to communicate that to some API level.
There been any changes spec-wise?

Speaker 4: 00:33:01

Yeah, I wish T-Bass was here.
I've been going back and forth on a bunch of things,

Speaker 1: 00:33:04

but I

Speaker 4: 00:33:04

guess I couldn't make it.
There were a couple of little things I thought I could ask while I was here.
The original splice command has the chain hash in it, and T-Baz was saying we don't need that because we have the channel ID.
I'm guessing there

Speaker 6: 00:33:15

was something

Speaker 4: 00:33:15

to do with that.
Maybe, Ressi, do you know?

Speaker 1: 00:33:20

Yeah, well, originally, the open you needed to, because you didn't have an existing context, right?
You were like, I want to open one of these.
And in theory, it could be a different chain or something.
So yes, that is redundant in the context in any channel message, right?
I think, yes, no, I'm not going there and like a channel that has multiple, no, let's just.

Speaker 0: 00:33:45

Yeah, I don't know about y'all, but we actually ripped out Litecoin in the past release.
Like it was just there and had some hacks around it.
So maybe we were the last ones.
I don't know.

Speaker 1: 00:33:54

We may have ripped it out.
I don't know.
No one's

Speaker 3: 00:33:57

said it.

Speaker 6: 00:33:57

I think

Speaker 0: 00:33:57

for us, it was working for several years.
So we just, No one complained.
So we're like, we can wrap this thing out now.
Oh, yeah.

Speaker 4: 00:34:05

The other thing we were messing with was, now that we're getting close to finalizing it, we just dodged the post-splice reserve requirements problem.
And we kind of have to answer that now.
And there's a couple competing ideas of ways of doing it.
Let me find my notes here.

Speaker 1: 00:34:25

One second.

Speaker 4: 00:34:26

Right.
So one idea is treat it as a fresh channel, So all the reserve requirements is to reset as if it were a fresh channel The second one is follow the old channel reserve requirements Until the new ones post splice are met and then swap over to them And the third one is just drop the reserve requirements entirely And apparently T bass messed a lot with the second one and found that to be extremely complicated, according to him.

Speaker 0: 00:34:53

I can see that.

Speaker 4: 00:34:55

That's kind of an up-in-the-air question, really.

Speaker 0: 00:34:58

I guess Robert entirely just means implicit zero reserve, basically?

Speaker 1: 00:35:02

No, because this should fall out of the zero reserve channel type, right?
If you splice and you say, and this is the channel type I want the result to be, and that type is zero reserve, then you've got your answer.
But your restrictions are always like the minimum subset.
How does this work?
The most strict of all the splices that are possibly happening at the moment, including the one you're on right now.
So that falls out pretty naturally with reserve, like who's got the highest reserve?
That's our reserve.
It does allow you to do stupid things.
And if you're assuming 1% reserve and you splice in a hundred times channel, now you're like, you're kind of fucked because you're in reserve until that, but don't do that.
Yeah, like, you know, Like people can always do things that are stupid.
Yeah, you'll have to wait until.

Speaker 6: 00:35:50

SPLICE at 100x, just open a new channel.

Speaker 1: 00:35:54

That's right, exactly.
But I do think it dovetails pretty nicely with the zero reserve feature.
You would say, hey, I want channel type zero reserve, that's fine.
And then you would still have the reserve until that splice is confirmed and you're on that for sure.
But the nominal, like you will max, the worst case requirement of all the splices and use that applies to reserve as well.
I think that's pretty simple.

Speaker 4: 00:36:24

Are we allowing that kind of channel change on reserve requirements during a splice or is that happening somewhere else?

Speaker 1: 00:36:31

You should have a channel type in there somewhere, because it's the obvious thing to do.
For example, just because your existing channel doesn't have anchors doesn't mean that your splice won't have anchors, right?
You should be able to control that.

Speaker 6: 00:36:46

VICTOR BRODINKOVICH Yeah, so this definitely has implications.
Like, if that's the route that we're going, where splicing includes channel type conversion and stuff like that, then it does very much intersect with a lot of the dynamic commitment stuff.
And I know that we have talked about

Speaker 1: 00:37:01

that before.
GARETH J.
LEWIS So what's an opportunity if we don't do it, I think.
Right?
I mean, it's-

Speaker 6: 00:37:06

Well, I agree.
I'm not actually saying that you shouldn't do that.
What I do want to say about it, though, is that the way that we're currently planning to handle that with dynamic commitments is that the new, like the re-anchoring step, and this is where it kind of differs from splicing because like not all of your channel liquidity is available until the actual splice confirms.
And with dynamic commitments, the whole idea is like we're not trying not to go to chain with the re-anchoring step immediately.
It's supposed to be kept off chain.
And so at the moment, we only enforce the new channel requirements.
We enforce the new channel like constraints or whatever long before the actual UTXO turnover happens.

Speaker 3: 00:37:54

Does that make sense?

Speaker 1: 00:37:54

GARTH GREENWILLIAMS.
Isn't that the same?
So the splice rule is that you do the worst, the minimum subset kind of requirement, which in your case falls back to the same thing, I think.
If the new Splice requires you to have more something, then you have to have more something immediately as soon as you propose it.
I think that's true.

Speaker 6: 00:38:16

But I think the difference is that the dynamic commitment rule is not most restrictive.
Because it's not contingent on a bunch of different competing.
Because this requirement comes from the fact that you can have multiple in-flight splices.

Speaker 0: 00:38:33

Can you reject a splice as is today?
Is that in the protocol today?
Yeah.
Because otherwise, you might want to reject the channel type upgrade.
It could be a dog breed or something.
I

Speaker 4: 00:38:45

don't know, right?

Speaker 0: 00:38:45

If that is alongside of it.

Speaker 3: 00:38:49

Yeah, we

Speaker 4: 00:38:49

do it with the, we use the tx abort command, which is already there from interactive tx.
And it's kind of a general like reject.

Speaker 1: 00:39:00

So, so for, so if you're not insisting, oh, I see, yes, because you're, huh.

Speaker 6: 00:39:12

These proposals are very related, but also they have very key differences.

Speaker 1: 00:39:17

Subtle differences.

Speaker 0: 00:39:18

Yeah, I feel like we've been trying to mash them up for some time now, but like I mean, I think also just based on all the momentum splicing already has, I feel like it should just progress.
But if there is that magical cutout that like, you know, comes to someone with a moment of insight, We definitely entertain that.

Speaker 6: 00:39:33

The key difference here, and this only even applies to a subset of splicing use cases.
But if you're splicing out, you can actually just invalidate your final commitment on the previous UTXO and then just wait for the splice to confirm.
But the problem is if you splice in, then you're pulling in new inputs, then you can invalidate the transaction.

Speaker 1: 00:39:53

Because you can do that, we can't.

Speaker 6: 00:39:55

The input, and so you're like, yeah, exactly.

Speaker 1: 00:39:57

Exactly.
So if you're not changing anything, then you get more power, you make a lot more assumptions.
You can go, cool, this is definitely going to happen in a way that we can't because we're taking in random bullshit from the other side.
So yeah, that does give you a window that we don't have, which is kind of interesting.
So yeah, you don't have to wait for anything to happen because you can go, well, it's the only way this thing is going to exit, which is kind of nice.
Yeah.
Okay.
So is your plan to like, other than not waiting, presumably you will actually output it and it will be broadcast and will eventually happen, right?
But you don't care?

Speaker 6: 00:40:37

Yeah, yeah.
We had a thought maybe that like, if we wanted to co-op close the post Dyn committed channel at some point that we might close it off the original UTXO, but I don't think that's going to be good for anyone.
So I think the goal is that in all cases there is an eventual confirmation of that re-anchoring step.

Speaker 1: 00:40:57

Well that's where that's where splicing already has that because our closes will apply across all of them.
So in theory, you get that for free because we have to.
We have to close all of the in-flight splices just in case one of them goes through.
So our mutual close code already covers that.
So you might get that one, which would be nice.

Speaker 0: 00:41:17

One of the distinction I think worth mentioning here, I think there's two classes of what we're looking at in the requirements.
One is where you have a kickoff that you never want to broadcast.
Right.
And this is the whole, like, you know, I went from basically basically say, what, you know, zero to basically be one now in the new type of channel.
That's one that has a little more special handling.
The other variants where it looks like we're going to the new anchor code and format basically, that would necessarily require the special handling.
That one can be combined a little bit more.
It's just the case of having the kickoff that you're not necessarily going to broadcast.
Certain occasions of that, for example, we're discussing, if you have that kickoff and if you do a few different instances of it, you need to revoke the old one.
Is it okay that you can just rely on someone broadcasting the second level so we have the second level HTLC?
I think that's where things start to diverge a bit.
I guess also some of the negotiation up front if for example, you're doing parameters like dust, you can't just say here's a new dust, maybe I want to have my own dust as well.
But it feels like we'll end up with 60 percent shared module here somewhere and maybe there's just like two different interfaces I'm not really sure how it's going to turn out in the end.
There's a lot more concrete code for slicing, so there's only that.

Speaker 4: 00:42:26

We're talking about just changing everything up.
What about adding some TX to the closed transaction?
Make that a split.

Speaker 0: 00:42:34

Yeah, that's a long-term feature.
I think I got bumped a week ago as well.
Yeah, that would make a lot of sense.
But I guess one thing at a time maybe.

Speaker 1: 00:42:46

Yeah, that one, as long as we use this parallel construction trick, where you both get to construct whatever you want, then it's really nice.
But again, it's got a 30% overlap with, it's not quite the same interactive construction, right?
Because I will let you add anything, whatever the hell you want to yours, and you will let me add whatever the hell I want to mine, because you don't care, which is kind of cute.
But it's different from interactive TX, where you're mutually trying to build a transaction.
You're actually trying to build two separate ones.
In the construction for closed case, it's slightly different.
It uses the simplified closed ideas just in interactive format.
So it's similar but it's actually not the same.

Speaker 0: 00:43:32

Cool, Okay.
Going down the list here.
So I think Eclair is working on a version of this.
So they asked some questions, just waiting for, you know, interrupt chance.
Nothing crazy has happened on mainnet still.
Some people are using it, some people aren't.
There was like a bug or two, but nothing shortstopper wise yet.
I think last time we talked about there's like a sec PPR up now for Music 2 stuff maybe that gets merged and like, you know, monthly trees and things like that.
I have to check that one in the later time.
Okay, here it is, yeah, it was opened January 6th.

Speaker 2: 00:44:12

How is user demand in general?
Are you seeing a lot of people asking for it?
It seems that the fee rates have calmed down a little bit since I think what we saw at like 600 or something stats per view byte this one time.
But yeah, because I'm thinking while fee rates are calm, people are probably not gonna be demanding it, but with 600 sats per view, even Taproot is probably not gonna see the sort of savings that people will be looking for.
At that point, we probably would need something like cross-input signature aggregation.
Do folks have

Speaker 0: 00:44:46

any thoughts?
I think that's one factor of it.
I think another factor is you can say like, people just wanting to run lazy.
I think Zeus is now looking to make a default, or at least if it's available for the user.
But I think the bigger thing to me is just like the progression of the top channels to gossip to PTLC.
PTLC gives us a bunch of stuff as far as privacy improvements, and also things like the form-hole attack, which I think we were looking at some other variants of it.
It's a means to the end to basically get there.
I think we have the path of basically doing the different PTLC that was ECSA and Schnorr, but we didn't do that.
We committed to the Schnorr roadmap.
I think to me that's the end goal there.
But I think primarily it's used by MobileWallet stage because obviously you don't need to run it in purposes otherwise.
We do have the gossip stuff as well.
We haven't given a code for it, but we pause because we just want to make sure we have good feedback.
Because we would literally make LND net if we just released our own gossipers.
We definitely don't want to do that.
So to me, the end goal is getting to the DTLC land and everything else is a stepping stone in between that.

Speaker 2: 00:45:49

Fair enough.

Speaker 1: 00:45:49

And I like that it's got cool points, right?
That's giving it a bit of a tailwind.
Like people are like, oh, I want taproot, why?
I want taproot.
It's like, okay, dude, cool, yeah.
I agree with you.
Like the payoff is further down and it's less like instant gratification for users, but it's nice that they go, cool, Taproot, that's nice.

Speaker 0: 00:46:06

Yeah, and that's our Clare should have recently where they're doing like a Taproot, a variant swap.
It's like they also figured out Miniscript stuff with that as well, where they have a way where someone can import a backup type of thing.
I think there's a tooling thing there as well, Not to say that it can't work, or maybe it can work either way, but I haven't dug into it as much there.
But yeah, fees are down now, but hey, who knows, who knows next month?
Things are, Things are all top.
Usually, yeah.
Like

Speaker 1: 00:46:35

a fucking goldfish.
It's like, oh no, fees aren't there anymore.
But wait, hold on, you were freaking out like five seconds ago.
Yeah, exactly.
They'll be screaming again soon, don't worry.

Speaker 0: 00:46:45

Yeah, exactly.
And then we're working on in Diatina just general like just improve feed bumping Basically, so it's like number one delay base and number two also get also getting ready for it's also getting ready for it's also getting Ready for it's also getting ready for it's also getting ready for some of the new code format changes as far as like not Sweeping anchors together things like that.
So we're so like we are working on that stuff I'm for it because you're working towards, you know a winter, you know, maybe comes and goes

Speaker 1: 00:47:05

Did everyone else just get some roast beef wrap

Speaker 0: 00:47:07

in there?
Yeah, yeah, yeah.
Getting ready too.
Wait, what happened?

Speaker 4: 00:47:11

It's a glitch, the matrix.

Speaker 0: 00:47:13

Oh. Oh. Okay, a remix?
Okay, all right.
I saw you on the active.
Did I say something funny?

Speaker 2: 00:47:21

Yeah.
Yeah.
Also getting

Speaker 4: 00:47:24

ready.
Like,

Speaker 1: 00:47:25

also getting ready to life.
Five times.
It just, it just kept going, man.

Speaker 0: 00:47:31

That's funny.
Cool.
But yeah, gossip stuff.
We have some code.
We're just basically waiting on some additional feedback from other people because we have to do all the same thing for this one to have a single network.
So we're just chilling on that.
I think since then, Elizabeth started working on supply path stuff.
True or errors.
I know this progressed a bit.
I know Tobias and Yoshu are going back and forth.
I don't know what the latest is here though.
But maybe I'll like write down to have updates ready for next time.
Boom.
Peer stories back up.
We have someone working on this now.
There's an issue with PR just for the wire messages.
I think there were some comments in the actual PR around like messaging and stuff like that.

Speaker 6: 00:48:22

Cool.
Cool.
Cool.

Speaker 3: 00:48:24

Nothing right there.

Speaker 0: 00:48:31

Offers.
Actually, we covered a little bit earlier with the multi-format for node identifier.

Speaker 1: 00:48:39

As far as I know, Matt's doing the human readable enhancements, which is open a bit of a can of worms for like how do we handle, you know, unauthorized ranges, you know, and there's the SCIDO pubkey stuff.
That's really the only thing that I know of.

Speaker 6: 00:48:57

Okay.

Speaker 3: 00:49:03

Okay.

Speaker 0: 00:49:09

Okay.
Chan, reestablish.
I think this one is just waiting on review limbo, just codifying some edge cases and how things work today.
Cool.
I guess last few minutes, anything else people want to chat about?

Speaker 1: 00:49:34

No, man, apparently we can't scale without trees that, you know, that OpCTV will give us and it'll all be magical and the exit costs.
There'll be insurance or something.
There'll be magical, the provider will pay the ridiculous exit costs and you won't have to, and it'll all work.
Stop looking so hard and asking such hard questions.

Speaker 0: 00:49:59

Yeah, yeah.
I definitely feel like we're in the phase where we just need to be building stuff with this stuff.
So they like sort of wrangle with exactly some of the issues are.
And yeah, I know the ARC people are working on something and they're working on something for a few months.
I think maybe they're running into this themselves now, but like, yeah, like you have our clothes and just even sort of like handling the different variants of how you get down there and close, new broadcast first, things like that.
I think we're in the same, people want the shiny new thing basically.
There's 26 shiny new things, now there's L2s all of a sudden that are not L2s.
I think we're in noisy phase.
I feel we can inject some more signal into it just with actual engineering work and some direct research and stuff.

Speaker 3: 00:50:38

It's the Craig Wright style of argument.
Look, there's a million documents that prove I'm Satoshian, that this is the best possible soft fork we can do, best thing since sliced bread.
Don't look at any individual one because most of them are forged or just kind of face plant.

Speaker 1: 00:50:56

But the sheer number is impressive.
Yeah.

Speaker 3: 00:50:59

But the sheer number

Speaker 1: 00:51:00

is impressive.

Speaker 3: 00:51:00

It's a

Speaker 1: 00:51:00

wheelbarrow full of improvements.

Speaker 3: 00:51:03

Oh, I forgot about the wheelbarrow.

Speaker 1: 00:51:07

Yeah, so, yeah, this debate has been really annoying, annoying me recently, and I think I'm probably not the only one, but wiser people are shutting the fuck up.
And that is almost becoming a problem at this point.
Because they're like, see, there's no disagreement.

Speaker 0: 00:51:23

And I'm like, ah.

Speaker 1: 00:51:24

It's like when you got one of those trade-offs, you found something cool, and it has these trade-offs, and you're really in denial about how bad that trade-off is, And you're really trying to cope badly with that trade.
Cause you like this part of the trade-off, but you're trying to like not pay that bit.
And you just haven't figured out how to do it, but your hopium is just, you know, off the charts.
It makes it annoying to discuss.
So I'm glad that people are going

Speaker 0: 00:51:51

to the math.
It does feel like we need like a scaling Bitcoin, we don't call it scaling Bitcoin, just where people can show up and have like serious discussions and possibly like done their homework ahead of time versus like the sort of like scattered thing, you know, things are happening right now.
But it feels like people could come together and just sort of like, you need to be this tall to get on the right kind of thing.
And let all of the noise blow away and people actually have their own experience around this stuff.

Speaker 3: 00:52:12

Part of the problem is to become, quote unquote, educated on it.
You have to go read like 40 different proposals that are composed

Speaker 1: 00:52:19

of different designs.

Speaker 3: 00:52:21

The problem is there are too many of them.

Speaker 1: 00:52:23

And there's some clever stuff in there so it's really hard to canonically say no that is definitely impossible because you know it's possible that deep in there somewhere has actually figured out a decent way of doing this with different trade-offs.
You can't be entirely certain that you have, because there may be the 41st document that you haven't read.
Right?

Speaker 3: 00:52:41

Right.

Speaker 1: 00:52:44

Yeah.
But yeah, maybe the Austin BTC++ scripting one is going to be the place we can, you know.

Speaker 0: 00:52:52

Yeah, looking forward to that.
I might stop by just randomly now, looking at it, May 1st.
I don't think I'm doing anything.

Speaker 3: 00:52:58

So it

Speaker 0: 00:52:58

could be cool just to be around where actual discussions are happening.
Or relevant ones at least.
Cool.
Okay, With that, thanks everybody.
Great meeting.
See you all on the internet and stuff.
Posted my notes and Carla has transcript which will go up afterwards.

Speaker 1: 00:53:18

Cheers.
Awesome.
Thanks.

Speaker 0: 00:53:21

Thank you.
Bye-bye.

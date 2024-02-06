---
title: "Lightning Specification Meeting"
transcript_by: carlaKC via tstbtc v1.0.0 --needs-review
tags: ['lightning']
speakers: []
categories: []
date: 2024-01-29
---
Speaker 0: 00:00:01

So I see that you all have made a few comments on the symbol close feature, so we'll cover that soon, but I just had one item on the dual funding that I wanted to highlight.
So on dual funding, I wanted to add one more thing before it goes in.
So right now, the CREA dual funding proposal forces you, whenever you add an input to the interactive TX transaction to provide the whole previous transaction for that input.
This is actually an issue in practice because that means it's restricted to 65k bytes because lightning messages are restricted to 65k bytes and actually whenever people are trying to do a kind of to bring funds into a channel from a coin join or from a mining pool payout then it usually exceeds the 65 kilobytes so the inputs cannot be used for dual funding which is really annoying.
So I'd like...
Yeah it's true that Rosby's keyboard is really strong, but...
So I'd like to get rid of that.
At least with Taproot, we know that the Taproot signature covers more things than the SegWit v0 signature.
So with Taproot, we said that we could remove this requirement.
But then I can't even remember why we had this requirement in the first place.
I remember that at some point Lisa and Rusty mentioned an attack that using SegWit v0 if we did not transmit the whole previous transaction this could be exploited and it looks like the issue for that hardware wallets have where you can force a hardware wallet to sign two different versions of a transaction and end up overpaying a lot of fees to miners.
But I don't see how this specific attack would apply to dual funding.
So if anyone has a clue why we have to provide the whole previous transaction when adding segment v0 inputs to a dual funded transaction.
Yeah, it would be useful.
Does anyone remember why we need this, why we need this and what attack can be made if we don't have this?

Speaker 1: 00:01:58

That's a good question.

Speaker 0: 00:01:59

I think

Speaker 1: 00:01:59

it was some like hardware wallet thing, like you were saying, you know, pre-TAP route basically, because now like, you know, say you guys all have to assign all the inputs as well.
That's a good question.
I mean, you know, random alternative is that like, you know, you can also just like paper over the limit just by actually doing application level checking.
So that's what we do for the ACID stuff on Gossip, or just do the old transport thing that we did and then removed.
And that you can have an application that's just spanned over many transport messages just by continuing to read on the application side.

Speaker 0: 00:02:26

But that's probably something else.
Yeah, but why would we do that?
It's really inefficient to transfer potentially, yeah, 200 kilobyte transaction.
If we don't have to, let's just not transmit it at all.

Speaker 1: 00:02:39

And I'd like to see if we really need

Speaker 0: 00:02:41

to transmit it because I think, I'm not sure we really, at least we know that for Taproot, we didn't see a reason to translate it.
Do we actually really need to translate it before taproot or is it something that we I don't know, all inputs are segwit, tx, but malleable in what way?

Speaker 1: 00:02:58

But yeah, well, I think he's saying that you want to verify that the inputs are set, but if you send the preval, you can verify that basically.

Speaker 0: 00:03:03

Yeah.
Yeah, exactly.

Speaker 1: 00:03:04

So that's just the preval instead of the entire transaction.

Speaker 0: 00:03:08

Yes.
I don't know why, but at some point initially we said we would only send the preval, and then Lisa, and I think Lisa mentioned that Rusty mentioned an attack and we had to send the whole previous transaction, but I couldn't find any reference to that and I don't see how the attack on hardware wallet would apply here.
So I don't see why we would have an issue with sending onLegalPriv out.

Speaker 2: 00:03:29

Well, Rusty just got here, so maybe we can ask him.

Speaker 0: 00:03:31

Oh yeah.
Hey Rusty.
Do you hear us Rusty?

Speaker 3: 00:03:41

Sorry

Speaker 4: 00:03:44

I'm making coffee so

Speaker 3: 00:03:45

I'm got

Speaker 4: 00:03:46

the sound

Speaker 1: 00:03:49

effect.
Yeah I think otherwise Prevalt seems like enough less bytes to be sent across the wire as well.

Speaker 0: 00:03:57

Then it's really an interesting...

Speaker 1: 00:04:00

What would you even do with that entire transaction beforehand?

Speaker 0: 00:04:04

Yeah, right now we don't validate anything more than we would validate with a preval.
So that's why I wonder, well, why did we do that in the first place?
And I can't remember.

Speaker 1: 00:04:15

Yeah, I mean, it's not like you can verify unspentness without like an actual index.
So it's just sort of what they had at the point.
Yeah, I don't know.

Speaker 0: 00:04:24

Yeah, maybe somehow verifying that the other party is not including inputs that are yours, but even if they did so with a prevout, you would check it all the same.

Speaker 1: 00:04:33

I don't see how- The input there's by the TX ID.
So even if like the output was repeated somewhere in the chain, that wouldn't necessarily do anything.

Speaker 0: 00:04:41

Yeah.
Rusty, do you remember that?
Because the only reference I had was Lisa telling me that you thought of an attack and we had to include the whole previous transaction.
So the context is for dual funding for interactive TX, the fact that we include the whole previous transaction instead of just the prev out.
I think

Speaker 3: 00:05:05

the case was where you, if you don't know the parallel, it hasn't already been mined.
I think you need the whole master.

Speaker 0: 00:05:16

But why?

Speaker 1: 00:05:17

If it's unconfirmed?

Speaker 3: 00:05:18

Yeah, If it's unconfirmed, if you haven't seen it yet, that was...

Speaker 1: 00:05:23

But shouldn't you require confirmation, I guess?
No, not necessarily.

Speaker 0: 00:05:29

Not necessarily, but...
But It seems to me

Speaker 1: 00:05:31

to accept zero-conf, like a zero-conf inferred for dual funding that's unconfirmed, like the entire thing can be invalidated, right?
Like the dual funding transaction.

Speaker 0: 00:05:37

Yeah.
Yeah, but it's okay.
You're going to detect that it's invalidated at some point and double spend it.
But even so, even if it wasn't confirmed, what more would you validate by having the previous transaction than what you would validate with just a preval and outpoint?

Speaker 3: 00:06:01

I would have to check.
It does seem weird.

Speaker 0: 00:06:08

Yeah, I haven't checked your code to see if you were actually validating things inside the previous transaction that you couldn't validate with only an out point and a prev out.
But Lisa is not here.

Speaker 4: 00:06:21

So yeah, I'll look in

Speaker 3: 00:06:22

the code and see if I can find what checks we're doing.

Speaker 0: 00:06:26

Yeah, that would be interesting because if we're able to get rid of that before we merge the spec, that that's a backwards incompatible change, but that's the last time we can do it.
So we should definitely do it now.

Speaker 1: 00:06:38

Yeah.
So like I like did like a Bitcoin.xyz search or whatever.
And I think it's kind of like the hardware case, basically.
Like an offline wallet, there's no transactions fees.
So you send the entire thing and then it gets compute the fee using that.
But like, it feels like it would still need to know the transactions all the way back to a point where it did know the full input value.
I think it's a hardware wallet thing, but it doesn't necessarily apply.
In a sense for Taproot.

Speaker 0: 00:07:01

It doesn't apply here because you are providing the previous TX for your input, not the other guy's input.
Here with the hardware wallet, it's a bit different.
So the hardware wallet attack is not the same because you are sending data to the hardware wallet for an input that the hardware wallet owns and will sign.
But here in the interactive TX, you are sending input details for input that you are going to sign, not the other guy.
So the other guy doesn't care that you're showing more data.

Speaker 1: 00:07:28

Yeah, I think it's okay Also because like, you know, the SIGHASH by default includes all the input values now, right?
So, SIGHASH all at least.

Speaker 0: 00:07:37

Oh, Lisa is here.
Perfect.
Hey, Lisa.
We have a question for you.
I don't know if you remember that, did you see what we were talking about, the previous TX and interactive TX and why we actually need that previous TX and not just an outpoint and a prevout?

Speaker 5: 00:07:59

Yeah, I believe it had to do with them lying about whether or not they're actually committing to a SegWit output.
So there's a possibility that they could tell you that they were signing for a non-SegWit output and basically add an input that isn't non-SegWit.
Does that make sense?

Speaker 0: 00:08:18

How could they do that?
Because the signature would still commit to the script pubkey, so it would still commit to the fact that it's using SegWit v0, right?

Speaker 5: 00:08:28

There's a reason.

Speaker 1: 00:08:30

I think this is for the unconfirmed case, right?
And that if it's unconfirmed and they did have mixed inputs, they could alter the TXID of the input.
But if it's confirmed, you don't necessarily need to worry about that.
Because at that point, the TXID is stable.

Speaker 0: 00:08:43

Yeah, but if they alter the TXID, their signature just becomes invalid.
And what did they gain?
How can they exploit that?

Speaker 5: 00:08:51

What about wrapped segwit inputs?
That, no, that would be stable.
The input wouldn't change, that's fine.
Am I still online?
I don't know.
There was a...
Let me go back and look, because I definitely thought through this a few years ago.
And one of the...
I mean, it's been changed for Taproot because I think it was the same reason that like a...
The same reason that a hardware wallet is asked or requested to sign whenever you ask a signature from a hardware wallet.

Speaker 2: 00:09:24

But I

Speaker 0: 00:09:24

don't think so because I don't see how this attack would apply because for the hardware wallet, It means that you give a sign for previous TX.
Here is the opposite.
You are giving the previous TX when you are the signer, not the other guy is signing.

Speaker 5: 00:09:38

So. Your signature doesn't commit to, your signature doesn't commit to the script pub keys of other inputs, does it?

Speaker 0: 00:09:44

Of the others.
Yeah, exactly.
So maybe

Speaker 1: 00:09:46

that's related.
It completes to the script and the amount now, post-segwit.

Speaker 6: 00:09:53

I thought...

Speaker 0: 00:09:54

Segwit v0 doesn't, I don't

Speaker 1: 00:09:56

think so.
Post v1, that is.

Speaker 0: 00:09:59

Oh yeah.

Speaker 5: 00:10:00

Right, which is why for Tapwrit, we wouldn't need to provide the previous TX, but you need the previous TX in order to confirm it's not.

Speaker 0: 00:10:07

Can you hear Lisa?

Speaker 4: 00:10:08

She's listed muted for me and I can't hear her.

Speaker 1: 00:10:11

I'm here.
Yeah, we can hear Lisa.

Speaker 0: 00:10:13

Yeah, we

Speaker 3: 00:10:14

can hear her.

Speaker 0: 00:10:18

Can't hear me.
I still don't see how it could be exploited.
The fact that there would be this somehow confusion.

Speaker 6: 00:10:25

Someone can give you

Speaker 5: 00:10:26

an input that is not a SegWit input.

Speaker 0: 00:10:31

But I don't see why because then when they sign...
Can you verify that

Speaker 1: 00:10:35

just by giving the previous out point?

Speaker 3: 00:10:37

But then they malleate the opening transaction, right?

Speaker 1: 00:10:40

But I think T-Bag is saying that's okay because you'll detect it or something like that.

Speaker 2: 00:10:44

Okay.
Yeah, but now

Speaker 3: 00:10:46

your funds are stuck.

Speaker 0: 00:10:50

Oh, no, I think that's the issue.
Okay.
Yes.
Okay.
They would tell you that they're using segwit and you fit, but actually it's not, and you, and the signature would then not bind to the script key that they give you in the prevout, so they would get away with it.
Okay.
I think that would make sense.
Okay.

Speaker 5: 00:11:07

Right.
But the signature scheme with taproot, we can definitely get rid of it.
The signature scheme with taproot fixes this whole, so to speak.

Speaker 0: 00:11:16

Yeah.
So I'd like to at least get a taproot exception where we don't send the whole previous TX.
So that's why I have this PR open and there are potentially multiple ways of doing it.
But yeah, I'll try to write down the whole attack tomorrow to make sure that there is an attack actually and there is a reason to keep that previous TX for the SegWit v0 case.
Okay, so I'll try to summarize that and I'll send it to the list.

Speaker 5: 00:11:42

Cool, thanks Patrick.

Speaker 0: 00:11:42

How is that different from just double spending?
The difference is that you are going to create an interactive TX where the other guy has an input that is not using SegWit.
So that while it's unconfirmed, I guess if you do zero-conf, then they could potentially malleate and the funding takes later.

Speaker 5: 00:12:01

Well, they could...

Speaker 0: 00:12:02

And blackmail you for the funds?

Speaker 6: 00:12:04

They could get a funding transaction

Speaker 5: 00:12:06

that's malleated then, right?
Like your funding transaction isn't stable if they use a non-SegWit input, right?

Speaker 0: 00:12:11

Oh, yeah.
So they would malleate it before confirmation, so your channel is based on nothing, right?
And your commit transactions are based on nothing.

Speaker 5: 00:12:19

Right.
That's the real problem here is that the funding transaction ID might change just because they put...
Like if they end up having to put signature data in the script seg, then the final TX ID isn't going to be what you expect it to be.

Speaker 0: 00:12:32

Okay.
And then they blackmail you to get back your funds.
Okay.

Speaker 5: 00:12:35

Right.
Or, yeah.

Speaker 0: 00:12:39

Okay.
I think I see.
I'll try to summarize that tomorrow so we can move on to the next topic.

Speaker 5: 00:12:45

Cool.
Thanks, Bastien.

Speaker 0: 00:12:47

All right.
Thank you.
The next topic is simplified mutual flows and I see that both Kegan and Laolu put some comments in there.

Speaker 1: 00:12:56

Yeah.
Something that was just sort of like, you know, miner reaching randomization.
Probably the only thing I think is the whole operaturing thing around Bitcoin D, like not really letting you use a non-zero value upper turn at a certain point.
To me, I don't think, I've never coded a case where we'd add the upper term, but I think it's sort of like a case where individuals want to like not have a dust output and go to miners, but in that case, like we'll need to mine this, because otherwise it can be non-standard now, or at least, I guess, I think not a standard, but wouldn't be accepted by the default API after Bitcoin 25.
So therefore, we don't want to require that just to be zero.
The other thing I noticed with implementation is that, to me, it wasn't always clear, cases where all three signature bearings would actually be wanted, basically.
In that, like, yeah.
So I think we need two, or I think we need two or one versus all the three different cases.
Because it feels like in certain cases, it's like they can just not make an offer if they can't pay for it or something like that.
That was the only one of the other things that I was there.
I need to work more on some of the restart cases basically.
Also, we were moving around some of our logic with the, we did a big thing to basically fix issues we had with the old QWAP close, where before L&D, just basically would reject a QWAP close if we had anything pending.
Now, we're doing the whole, okay, it's going to be flushing, blah, blah, blah.
And basically the whole slow-mo kind of a thing.
So now we have that.
So I think that caused to put her on in this other area to make sure that things were compatible between our version and that version, which is the version you can say.

Speaker 0: 00:14:16

I think we need all three types of signatures and there's always one case where each one of them is used.
But it's true that the spec tries to be exhaustive and cover all the potential case even though I agree with you that in some cases you will just not send closing complete because you don't care.

Speaker 1: 00:14:33

Exactly.
Why would you send closing complete with no close or closee?
You're initiating the close.
At that point, you just wait for the other party to do it.
Or the idea that you always want to base force a new transaction or a new closed signature to be obtained, even if you can't pay for fees yourself.
But I thought the point was that closing complete, you're paying for the fees, but then you can pay for the fees, but remove your output.
I guess that's...
And if you're removing your output and you're paying for the fees, why are you trying to close it all?

Speaker 0: 00:15:01

You're altruistic.

Speaker 1: 00:15:03

I guess that's the thing.
Whenever I get into like an altruistic, you know, if clause or something, I'm like, like,

Speaker 2: 00:15:11

that's actually very similar to the comment that I just left too, which is that like, We have this desire to remove the co-op close, the original one in favor of the simplified one but if you have a scenario where the like Alice opens a channel to Bob but doesn't actually move any funds in that direction And Bob wants to close that channel for whatever reason.
If we don't have the old co-op close flow, then Bob can essentially, even if he wants to cooperate with Alice, allow him to cheapen the fees for Alice, he can't.
He basically is like, well, I guess I'm just going to go offline and now you have to force close.
It's like, I would have helped you out.
I'm like, I can't now.

Speaker 0: 00:15:49

Yeah, but you can send shutdown.
You still send shutdown so that you notify that the channel is now closed and then it's Alice's job to finish the signature and you can forget almost all about it.
You just need to stay online to be able to give her a signature.
But apart from that...

Speaker 1: 00:16:04

But yeah, you're talking about a case where the initiator of the co-op co-host has no funds in the channel at all.
So you're saying like they're sort of stuck.
Is that what you're saying, Keegan?

Speaker 2: 00:16:11

They're not stuck.
I mean, if they have no funds in the channel at all, they can just abandon it.
What I'm saying is that even if they want to be altruistic or like cooperative with their channel peers, like I guess that this comes down to like what Bastien is saying is that maybe just because you're the initiator of shutdown doesn't necessarily mean that you're the one who has to send closing complete first.
And it's Alice's job.

Speaker 1: 00:16:34

Yeah, it's different now because before, the initiator needed to send a closing sign first.
But now that requirement's gone away because you can basically have these two processes running independently.

Speaker 2: 00:16:44

Is there no, what's the shelling point then of who sends closing complete first?

Speaker 1: 00:16:53

Both sides can send close and complete now.
Huh?
Yeah, both sides can send, like, this is something I drew up, I don't know if this is useful for anyone, but like this is slightly out of date for what I have now.
But like, so basically like once you send shutdown, either side can send close to complete because there's sort of splits into two different versions, right?
Where I say close to complete, you send closing or close, say, whatever it's called.
And then the other way around, you can do that as well.
And then either person setting shutdown basically takes us back to the top and we can redo the whole thing.

Speaker 6: 00:17:18

And that's just getting

Speaker 0: 00:17:19

the RBF.

Speaker 6: 00:17:19

Can you

Speaker 2: 00:17:19

actually tell someone copies the same fee?

Speaker 1: 00:17:23

So there's no negotiation.
It's just basically now like I'm paying this fee for this transaction and you say okay and then you can say I'm paying this fee for this transaction and I say okay.
In theory, you should always make sure that you're increasing and be RBF replaceable as well, but that's your version.

Speaker 0: 00:17:42

Yeah.
The way I implemented it in Eclair is that as soon as you've exchanged shutdown, if you have funds in the channel after creating the closing transaction, you send closing complete.
Otherwise, you don't send closing complete and the other guy is going to send it.

Speaker 1: 00:17:55

Yeah, I have the same thing where it's like, I have like should pays for, or it's like if you can pay for it, do it, otherwise not.
And there's probably more fancy heuristics you can use when you decide to drop the output versus not, but I just do the straight thing.
And you know, right now, I think for that last comment, I don't implement the case where I send close and complete, but I don't have an output, basically, so I send no close or closee.
I think in that case, I'll just wait for the other party, but like I'll probably enumerate that just to have the test coverage there, but at least I don't think we'll, you know, initiate a close and complete with that where it's like I'm initiating, but I can't pay.
So I send the SIG without my without my or like I can pay, but not enough to also have a non-Dust output.
I guess that's where you would use it.
And this is like the signature where you send a TX without your own output, but with the other party's output, but you're still paying for fees.
That was the one that sort of like, you know, I got hung up on a little bit.

Speaker 2: 00:18:48

Yeah, I've withdrawn my comments.
I misunderstood something about the rest of the way the protocol worked.

Speaker 1: 00:18:55

Yeah, yeah.
It's interesting in that like once you get past the shutdown phase, it's sort of fully, there's like two independent states going on.
So that was something I was modeling initially, where it's like, okay, you can just continue to send close and complete on this one, and I can do nothing.
So you could ask me for a signature, send shutdown again, and then ask me for a signature, and just keep doing that.
And you can do the RBF ratchet, and I can do nothing basically, because maybe I'm a concern, but you are.
But as long as I'm online, you can get additional signatures for higher fee or whatever.

Speaker 3: 00:19:20

Which is kind of cute, because you can also change where the shutdown's going if you want to.
I mean, best effort, right?

Speaker 1: 00:19:26

That's a good thing to make sure I verify.
Like the second time around, a front shutdown is still there.
One other thing is, in what case would you want to set the sequence to anything else but the default RBFable, which is sequence minus 1 or whatever?
Is that just because we can do it with log time?

Speaker 0: 00:19:45

Or?
And I'll describe it.
Isn't it for multi-facety sniping where there are potentially more complex ways to set nSequence and lock time?
But then we should also, I think we should also add lock time to this message.
We only added nSequence, but we should also let us configure the log time.
And then there's this beep for anti-phishing sniping that lets you do fun stuff, prevent sequence and then log time.
So I think the ability to do that.

Speaker 1: 00:20:13

Yeah, and I guess it's the other party's transaction, so you don't care if they time lock it for some demo.

Speaker 0: 00:20:20

If they

Speaker 1: 00:20:20

want it to in a year, it's like, okay, well, I'm going to get out now, ideally.

Speaker 3: 00:20:24

Yeah, we should definitely add that.
Yes, all the things, whatever you want.
If you want it to look like other things, whatever that happens to be next week, you can do it.

Speaker 1: 00:20:38

Cool.
But yeah, I mean, it should be actually ready for interop this week.
I went on like a bunch of side quests, just trying to like clean some stuff up in L&D, you know, the whole refactoring, that would hold, turn around some new ideas.
But yeah, so I'm pretty close there.
There's like some other PRs and the final one should go out this week.
And then I probably like ask on like IRA or something.
Someone has like a node so we can do some basic interop.
Obviously got to do the LND interop first, But that's like a little more straightforward to test in our system.

Speaker 3: 00:21:03

Yeah.
And if you want to patch the spec now, you've actually got some code, cause it's kind of like, you know, it's all head code at the moment.
Plus I haven't swapped it in for a month that I've been away.
Yeah.
That would be kind of cool.
We've got a release coming up, but after that, I'm hoping to get back to actually implementing this.
So if you've got something that I can throw against, that'd be good.

Speaker 1: 00:21:25

Cool, I think we're planning on like loftily including something like this in 18 and like maybe the whole staging bit thing before the root of force, like we'll do like bit 160, just because I think right now people can use all the feed tools they can get for a lightning node right now.
This is definitely something to like how your cop could be able to move forward, which is super important.
Also just remove the negotiation stuff.
Now it's just like 1.5 round trip.
I have a signature, you have a signature.
We can go part ways now.
Cool.
Then The other thing that I was thinking about is that people see this diagram thing here.
Is there any interest of including stuff like this in the spec, even though maybe right now certain of the terminology is very specific to the way I monitored it, but do people find this useful?
The other thing I have is, we have an ASCII diagram that has a certain process.
I have one that's basically using mermaid but it's like a flow diagram.
If you find that useful, because if so, I can like, and the nice thing is it's just markdown and GitHub will render it automatically.
Delving can render it as well.
I can comment the raw version of this for both those people who find it helpful.

Speaker 3: 00:22:32

Yeah, I'm more than happy to move beyond asking.
Okay, cool.

Speaker 1: 00:22:52

Oh, yeah, that's about me.
That's about it for me on the core code stuff.
And now I'll just move on to Gary for interruption and stuff like that.
Anything else that you want to add?
I want you to talk about it.

Speaker 0: 00:23:02

Perfect.
On my side, I think the only difference in my implementation and what is in the spec is that I added log time along with the sequence field to the messages.
But apart from that, I have an implementation of exactly the spec.
So I should be ready to do tests.

Speaker 1: 00:23:18

Cool, yeah, and I mentioned the other thing that we fixed a long time is that now LND will actually do shutdown, but before we just complained that we can't do it, so now we'll actually do it, so that should be nice for everybody.

Speaker 0: 00:23:29

Nice.

Speaker 1: 00:23:31

Thanks, Keegan, for that.
Long time coming.

Speaker 0: 00:23:36

All right, then next up is trampoline routing.
Last time we discussed trampoline and different ways of doing it.
I need to do another pass on the spec because I think I haven't touched it in two or three years.
But I think it should be, unless Arik correct me if you discovered other things, but we were probably going to stay with the onion in an onion construction, where that inner onion can be viable length, but we would probably want to have a default that everyone uses so that you look like everyone else is a trampoline on the network so that you don't tell people that you are this specific wallet when you are building trampoline onions.
We have implemented relaying to a blinded path through trampoline, which is actually quite easy to do.
So I don't know, Arik, what you've been working on since the last two weeks, and if you've made progress on that or other teams.

Speaker 7: 00:24:28

Yeah, I'm fully on board.
I haven't really found anything else besides what you and I have been talking about.
So it seems it's not really worth the rendezvous trouble.
So let's stick with the nested onion construction.

Speaker 0: 00:24:45

Cool, awesome.
Then I'll do another pass on the PR to make sure that there aren't things that are left empty wrong or have changed since the last two years.
And otherwise we can keep making progress directly on GitHub.

Speaker 7: 00:24:56

Yeah, and it's actually, it's already ready for interop testing.
If at some point you want to do that and then maybe come up with more permanent values for the TLV IDs.

Speaker 0: 00:25:09

Perfect, sounds good to me.

Speaker 7: 00:25:12

OK, awesome.

Speaker 0: 00:25:15

All right, so next is channel jamming.
I know that Carla wanted to discuss some Flag Day activation and had made some updates to the BIP.
So, Carla, the floor is yours.

Speaker 6: 00:25:24

Yeah, so last year we spoke about writing up details of running an experiment on mainnet, and I've done that in the book.
But I just wanted to ask if people feel like we should include a feature bit here, because we don't need to.
It's an odd TLV, but it could be nice to have for the sake of knowing when we can turn that flag day on.

Speaker 1: 00:25:50

Well, is the flag day that I'll relay the new if they add with the bit?

Speaker 6: 00:25:57

Yeah, and I'll set it as the sender, which is the more important piece because we want a significant portion of people relaying before you start setting it as a sender.

Speaker 1: 00:26:11

I see.

Speaker 0: 00:26:12

That does expose you as a sender, right?
Because if almost nobody activates that feature bit, then as an intermediate node, you see an HTLC coming in with a bit set.
That means the previous guy was probably the original sender of the payment.
So privacy-wise, it's not that good to use a feature bit here, right?

Speaker 1: 00:26:31

But I think you'd want that because in the sender can like seek out paths that will actually like give them this relevant information, right?
And, you know, assuming this is like a required thing in the future, that's what they would do, right?
So which I how today, like, you know, if they're not, if they don't support the legacy, if they support legacy onion, you're probably not gonna run through them anymore, because I think, I think we got to remove that, right?

Speaker 6: 00:26:48

Yeah, and I think the benefit of a feature bit is we can only start setting it as a sender when a very significant portion of the network is upgraded and then you don't actually out yourself.

Speaker 1: 00:27:00

And Carlo, you mean a feature bit in node

Speaker 0: 00:27:01

announcement, right?
Sorry?
You mean

Speaker 1: 00:27:01

a feature bit in node announcement, right?

Speaker 6: 00:27:03

Sorry?

Speaker 1: 00:27:05

You mean a feature bit in node announcement, right?

Speaker 6: 00:27:08

Yeah.

Speaker 0: 00:27:12

And so we'd use a high feature bit that we can then at some point just forget about and remove?
Is that your plan?

Speaker 6: 00:27:19

Yeah, exactly.
And then if we do end up rolling out protocol level, we could either just remove the experimental one or set both the experimental and the low for a while.

Speaker 2: 00:27:33

Maybe this is a little too like researchy, but like, would you even need a flag day?
Could you just like look at the, the overall like note announcements and see what percentage of them are relaying it and set the thing as the sender with some probability that's like scales with that.

Speaker 6: 00:27:52

We could, it's worth the code compared to just manually doing that once and hard coding it, but we totally could.
You could pick the percentage of the network that you are pleased with as a center?

Speaker 2: 00:28:04

Well, what I'm actually just saying is that whatever percentage of the network, like, let's say it's 50% of the network, then you set it 50% of the time.
It's like one division.

Speaker 6: 00:28:16

Oh, yeah, but you have to like go and check every node.
It's just more code, right?
Compared to the playing feature bit, once off checking the whole network, and then you would still set it with some probability for sure.
But I don't know if it's worthwhile.
Yeah, I don't want to write that code, exactly.

Speaker 1: 00:28:34

That's fair enough.
Other question is, after this, what's the main data collection method?
This is an experiment.
What's the source that we're gathering in?
Is that eventually senders will report if things were fully endorsed?
Or is this just people setting it, just in terms of thinking further to the experiment of when we start to actually get some information from this?

Speaker 6: 00:28:58

Yes so the experiment at least for Allen D is running Circuit Breaker alongside And I've already got some people running just like basic data collection.
And the idea is we collect all of these signals and be able to perfectly dry run the reputation algorithm on mainnet.
So with that signaling, So we'd be able to understand exactly how this would impact traffic or not.

Speaker 1: 00:29:22

And that's assuming like, you know, a good bit of routing nodes, running circuit breaker and reporting is basically right?

Speaker 6: 00:29:28

Well, as long as like, if everyone is relaying the signal, You don't need such a high portion running the experiment, right?
Cause everyone passively propagates the signal and then you get like a few large nodes like running and setting it.
Cause once they set that signal, if they said negative, it drops and it passively relays more.
And for them, that signal

Speaker 0: 00:29:48

is valid.
I see.

Speaker 1: 00:29:50

Yeah, and the idea is that circuit breaker is a thing that's conditionally setting this to negative, basically, right?
Because everyone sets it to 1, and then they maybe set it to 0 based on like, whatever happened.

Speaker 6: 00:29:57

Yeah, it drops it down to 0 according to its reputation thing, but no one takes any action on it.
And I know Vincenzo has been thinking about a similar thing, a plugin for CLN.
And because I've got all the reputation written in a library, he can use the exact same library.
So it's very little code, which is nice.

Speaker 0: 00:30:20

So what exactly do you need from us?
Do you need us to actually work more on setting that flag day?
Is there a call to action?

Speaker 6: 00:30:29

Well, It was more a question, should we set a feature bit?
I'm leaning towards yes, because I haven't written that up.

Speaker 3: 00:30:37

Cool.
I got that.
Yeah, we've been bitten before when we don't set a feature bit.
Set a

Speaker 1: 00:30:41

feature

Speaker 0: 00:30:42

bit

Speaker 3: 00:30:42

and add 100.

Speaker 6: 00:30:44

Yes.

Speaker 3: 00:30:45

So add 100 during deployment in case it changes.

Speaker 1: 00:30:48

Okay.

Speaker 6: 00:30:50

And then the other thing I just wanted to ask, cause this has come up a few times, this is written up in a blip, and I know that that's not really the right place for it, but I don't want to spend a lot of time bike shedding about whether it should be in the blips or the bolts because it is kind of a weird gray space.
So if we want to move it, I'd like to hear if anyone has strong opinions on that.

Speaker 0: 00:31:13

Blip is fine for me.

Speaker 1: 00:31:15

Okay.
Yeah, I think fine there as well.
Maybe, I mean, eventually you could graduate, assuming like this becomes, you know, the default basically, and we just learn more from it as well.

Speaker 3: 00:31:27

Yeah, once it's something that everyone should implement, you should put it in the bit.
But until then, it's, it doesn't really matter as long as it's documented somewhere.
Yeah, the problem is if you don't put it in the bit that you end up with clashes of like someone else uses the same fields or feature bits, whatever.
So we don't have a place outside the bits for feature bit reservations.
So you should probably put up a PR at least that reserves the feature bit in the title of the PR.
That's generally how we hand them out these days.
Just pick the next one that isn't in the title of something and that's yours.

Speaker 6: 00:31:59

All right.
Great.

Speaker 0: 00:32:03

Yeah, the BIPs also have a way of setting up feature bits, even though I think we say that it should only reserve high feature bits.
But then we can merge the blip in the blip repository, and that's one place people should look at when adding new feature bits to see that it is reserved as a bit.

Speaker 6: 00:32:23

Okay.
I'll, I'll add a feature bit and reserve it in the title.
And then it's sort of ready for us to start writing some code from.
I've, I've done L&D code.
It's not too bad.
Just copying one field when you relate to HLC.

Speaker 0: 00:32:34

So,

Speaker 6: 00:32:35

hopefully it's not too difficult elsewhere.
Cool, that's me.
Thanks.

Speaker 0: 00:32:44

Perfect.
So, next up is liquidity ads.
We've had some discussion before the end of the year about liquidity ads on the mailing list, on the REAP mailing list, and the latest state of those discussions were there was somewhat general agreement among the people who responded that the CLTV lease doesn't make sense, isn't useful at all, just doesn't do what we do, doesn't protect against what we want.
So I'm still in favor of not having any CLTV-based lease, But this is definitely up for discussion.
I don't know if people have thought about it a lot or if it's not on anyone's radar.

Speaker 4: 00:33:26

I mean, I was thinking about it.

Speaker 3: 00:33:29

Oh, Dusty, yeah.

Speaker 4: 00:33:30

I was just saying I've been thinking about it.
I don't have any grand insights.
I mean, just intuitively, it feels like having the lease on chain makes sense to me.
I was talking with Lisa about an option, and we were saying that one solution would be the lease would stay active until you receive a payment that puts you below the lease amount and that would automatically cancel the lease.
And I think that solves a lot of the problems.
We haven't formalized it.
I probably should write it out.

Speaker 1: 00:33:59

Why would you want that though?
Because like you're paying for the allocation of the inbound, right?
And even if your distribution shifts, you still have the potential to receive all of that.
And first of all, I dropped out of the conversation towards the tail end.
I think when people are analyzing different scenarios, I guess for me, I guess it just depends on what people think they're buying, right?
I think that's the main thing, basically.
And if it's the minimal that you want to at least ensure that the capital is allocated to that output for a period of time, using CLT or CLT or whatever else can give you that, that doesn't, notwithstanding the party just not being online or whatever else.
I think that's a different thing of sort of like, the capital allocation versus the service you're getting.
And I think those are two slightly different things.
So like, I would be in favor of keeping it, but I'm not like super involved in the discussion right now.
But if people think it doesn't matter, then it feels like it waters down a lot of what people think they're obtaining.
Because there's something where you can just have a sort of like service of agreement with some party.
Let's say it's a wallet, let's say it's an exchange or whatever else.
You can do that without anything protocol required.
You can do that today basically.
But if you want some sort of like enforcement there, It feels like it's an important component.
If you're saying, OK, this is predicated by, I'm making sure someone allocates this capital to me for a period of time.
But if that's not what you're selling, or people think what they're buying, then I guess you don't necessarily need it.
And you can just do a handshake on the side, basically, instead.

Speaker 0: 00:35:16

Yeah, I think I can.
Yeah, but it's still good to have this handshake documented and specified so that everyone uses the same way of advertising their rates, advertising the details of their liquidity lease, even if there's no enforcement.
Because I think what people are really buying is the beginning of a relationship.
When you buy a liquidity ad, you actually want the other side to keep that thing open for longer than the liquidity ad and to see that it's a healthy relationship for both sides and that they keep adding liquidity without you having to pay them all the time.
I don't think you're only buying one amount all the time.

Speaker 1: 00:35:52

If there's no lock-in, they can just recycle, right?
Before, at least like I put one BTC, it's locked for two months, right?
But now I can just recycle it.
So what are you paying for?
Okay, yeah, I mean, if you want to accept that, you can accept it.
But it just feels counterintuitive to sort of forecasting an opportunity cost and also pricing that time value of Bitcoin.
Because at this point, if there's no lease lockup at all, then what's the time value?
It's just me allocating coins because you paid me a particular price.
Okay, what's that price based off of?
How good of friends we are?
But to me, it's a very large product question that I think people haven't really answered yet generally.
There are a few different solutions, but they're not effectively being utilized yet.

Speaker 0: 00:36:33

But we put a lot of details on the mailing list, Fred, with a lot of those details.
It would be interesting to respond on that because that's where I think I showed that the seller is taking much more risks than the buyer here.
So that's why it's a bad incentive for the seller to look at that liquidity.
And they would just end up providing only offering very short pieces.

Speaker 1: 00:36:53

But that isn't the compensation that the seller is being compensated for that risk, which is the time value of Bitcoin that, you know, basically there's

Speaker 0: 00:36:59

no true compensation.
There's no way to price that correctly.
No, but that doesn't make sense.

Speaker 3: 00:37:08

There's a huge variance between a bad buyer and a good buyer.
That's the problem, right?
So a bad buyer just basically...
We were thinking of the case where they don't use the liquidity.
So you're just tying it up and it doesn't cost anything.
But the case that Bastien is worried about is that they open this huge channel and they push all the liquidity back out.
And then you're kind of stuck.

Speaker 4: 00:37:31

That's the solution we're working on, is if they ever push the liquidity beyond the lease, then the lease just cancels.
That should solve that problem.

Speaker 0: 00:37:40

That doesn't really solve it because you also have to handle potentially multiple parallel leases.
And if you add one output every time, then potentially the commitment transaction also gets big and that's another way to grieve the seller.
So it really gets complex when you start going down that road.
That's what we did on the main base and we detailed that solution and we did that it has real drawbacks for the seller as well.

Speaker 5: 00:38:04

My whole point, I mean, I think from a protocol design perspective, I think there's like, you know, you're pricing to some extent, you are pricing risk, so to speak.
And like your capital allocation, you know, you've got that time value.
One thing that we can do in the protocol, I feel like, is you can make, one of the things that the updated version of protocol includes is the option to change that lease time.
You could advertise that you've set that lease time to zero, if that makes sense.
Like if you as a seller don't want to make a lock time commitment, the protocol can it like, I feel like we've structured the existing protocols such that you can only accept zero length leases as a seller, if that makes sense, and that'd be the only thing that you're selling.
And I don't feel like that precludes other sellers from necessarily, you know, then you're competing with sellers that are offering a lease time, if that makes sense.
Like,

Speaker 3: 00:39:02

yeah, at risk of, risk of complicating the protocol, You could have two, you could have like the, this is the minimum, this is what I'm committed to.
And this is the number that, you know, in a happy case, I am prepared to offer.
So modulo abuse at one month or whatever, right?
Where abuse is defined hand wave mumble, you know, be a nice person or whatever it is.
That would be one way of doing it.
But it is a more complicated protocol and avoid and unfortunately makes it difficult to compare apples to apples.
But then, to be fair, it's always difficult to compare liquidity offers because it depends on who's offering.
Because as we've said, you can't enforce service.
So there's always some qualitative.
Now, from a protocol position, it is a lot easier not to have anything enforced, because that's a lot simpler.
But I'm reluctant to give it up entirely just from a high-level point of view, because I do feel that if you're buying from anons or relative anons, you kind of want some, then have some skin in the game.

Speaker 0: 00:40:11

Yeah, what would you buy from an anon?
If they're not well positioned in the graph, you have no reason to buy from them because it's inbound liquidity that doesn't bring you anything.
And I would say that.
That's the problem with the buyer.

Speaker 1: 00:40:23

Right.
Like, when you go to buy a used car, you want to get a report, you know, as far as the mileage and things like that.
Right.
If you buy a used car that has 200K miles, that's your fault.

Speaker 0: 00:40:35

It's kind of different.
Buying inbound liquidity from someone who is already at the edge of a network is not going to bring you actual inbound liquidity for payments.

Speaker 1: 00:40:42

You're correct.
So you shouldn't buy that.

Speaker 3: 00:40:43

You can tell that.
Yeah, you can tell that.
Don't buy it.

Speaker 0: 00:40:46

But exactly.
Yeah, exactly.
But if nobody is going to buy it, why do we optimize our protocol for that use case and make it really more complex?
Because what I'm afraid of, try implementing that least solution with CLTVs. I tried it.
It has impacts on everything because it impacts force closes, it impacts the commit transaction format.
I did it.
It's really complex.
It's really annoying.

Speaker 3: 00:41:07

That's why we used CSV originally.
We just

Speaker 1: 00:41:09

hacked the CSV

Speaker 3: 00:41:11

for a very good reason.
We did not want

Speaker 1: 00:41:12

to change it.
I think the solution is to yell around the leak, but the leak can be patched just by reducing the number of in-flight HTLCs, and then also the max allotted amount as well.
And in the future, we're going to update those values dynamically.
You can control that if you really wish to.

Speaker 0: 00:41:25

Yeah, but your solution is subject to the attack that I described in the mailing list, that if someone pushes a lot of liquidity to you and then goes offline, you have a lot of liquidity that is locked for a very long time and you're just screwed.
CSV or CLTV, you're just screwed.

Speaker 1: 00:41:36

But you're not screwed because you got paid for it, right?
If you're the seller.

Speaker 0: 00:41:40

Yeah, but you get paid for a much smaller amount than what is actually locked.

Speaker 3: 00:41:44

Yeah, You would not let them open that.
You'd say, no, no, if I'm selling you liquidity, you're supposed to be buying my liquidity.
If you've got more than half the channel, I'm not even going to let you open it.
Right.
I mean, that's perfectly reasonable to say, no, you can't open.

Speaker 0: 00:41:58

But then you're missing out on real opportunities as well, because if later that guy just wants to splice in some funds, you should not let them by your reasoning, you should not let them splice in those funds, you should deny that.
But if they're honest, they're going to splice in some funds, they're going to push that out, which gives you routing fees for the segment that is going out and it's all in your favor.
So an honest guy, you would want them to do that.
But a dishonest one, you definitely don't want them to do that, but you're missing out on a lot of revenue if you're blocking everyone just because someone may be malicious here.

Speaker 4: 00:42:28

Well, there's a middle ground that if they push past the lease amount, it just cancels the lease automatically built into the protocol.

Speaker 0: 00:42:37

Yeah.
Actually, try implementing that, because it creates a lot of edge cases.
Because of a parallel HTLCs, things could update, can be going in both directions at the same time.
This is going to create a lot of false closings.

Speaker 2: 00:42:49

Yeah, I think I saw it suggested in that thread where you originally proposed this that like maybe you can go about creating this like counter where it's like there you only have I know that you said that parallel leases might make this really complicated, but I mean, is there a case where we really think that parallel leases with different terms is useful?
Or do you think it's just like, okay, we might want to add more to the liquidity lease, but we don't want to have them like heterogeneous terms.
Like this one's locked up for two weeks and this one's locked up for four.

Speaker 0: 00:43:23

It's rather that if you have an ongoing lease and you actually want to buy more, it becomes really hard to figure out what you're actually paying If you are trying to merge the new buy with what's already existing.

Speaker 1: 00:43:34

You can, it can just be a new channel and it's segmented, right?

Speaker 0: 00:43:39

Yeah,

Speaker 1: 00:43:39

but I know everyone likes the single channel.
Yeah.

Speaker 3: 00:43:44

No, no.
The Lease is definitely a per channel thing, not a per input thing.
That's that just gets insane.
You can't do that.
But the point is that if, you know, general, generally like hiring, like, so, so leasing liquidity is, a, okay.
Okay.
Well, this is good.
So they want to buy more liquidity from you and they want to in the same channel.
That's, does it refresh the whole thing?

Speaker 2: 00:44:11

I think that that's the only sensible way to handle it, right?
Is that any new leases that you want to take out from the same provider on the same channel should probably just cancel the old ones.

Speaker 3: 00:44:22

Well, yeah, you don't want to have multiple leases in parallel.

Speaker 1: 00:44:25

You can just roll over and combine, right?
Like we drew some stuff up like that.
We just never implemented it because like people didn't really care about this stuff then.
I think like in the future they will, but things are still not that mature as far as like capital allocation wise.

Speaker 3: 00:44:39

The problem is as T-Best points out, then you have like, so do you get the new lease terms as soon as you start the splice or when does that happen?
Or does that because you've got multiple ones in progress, do the old ones apply the old lease?
The new ones apply the new lease?
I guess so.
And that's a bit of a nightmare to to organize.

Speaker 0: 00:44:58

Yeah, there are a lot of such cases like that where it becomes really complex when it starts interacting with everything we already have in an implementation.
So I'm sticking for now with the simplest implementation.
If someone wants to implement the stronger version and thinks it's okay and still worth pushing to the spec.
I'm waiting for it.
But honestly, this is a lot of really amazing.

Speaker 1: 00:45:21

We've implemented the CLTV version.
The only thing we don't have

Speaker 0: 00:45:24

is the...
You all projected that attack.
Yeah, I can use that attack.
No one is attacking it right now.
No one

Speaker 1: 00:45:29

is doing that right now.
I think the distinction is that I'm approaching it primarily from the point of view of bootstrapping inbound for a merchant to receive or otherwise.
And I think in that scenario, it's a lot clearer what you're buying.
You're basically buying the potential to receive capital in the future.
And you don't necessarily care about the distribution because you can do whatever you need, swap for whatever else, like regain that inbound, but you bought that connection point basically.
But I think you have a slightly different model of like, you're buying the inbound and the outbound connection point, but like mine is like, you just have that inbound

Speaker 0: 00:45:57

connection point.
Not really.
In that case, I don't see how that applies.
It's just that in your case, so I'm a merchant, I'm buying from you so much in the channel, then I place in some funds, I pay that and they disappear and potentially you have locked up in that channel for a very long duration.

Speaker 1: 00:46:16

I got paid though, right?

Speaker 0: 00:46:18

Yeah, but for what duration?
If the duration is six months, you get paid once for something that is going to be stuck for six months.
Is it worth it?
How do you price it to make sure that it's worth it?

Speaker 1: 00:46:26

Well, so there are a few different versions, right?
Like, you know, one in that like, splicing is the only way to get funds in and out.
The other thing around the duration as well is that you don't necessarily have to pay it as a lump sum basis.
If you develop another unidirectional payment channel on the side basically that uses time lock to basically pay out those coupons to an individual, you're not necessarily...
That's better for the buyer because the buyer is only paying for the duration in which they received it.
But I think it's a difference on the way you're viewing what you got paid for.
You think that you're getting short circuited or you're getting short changed if for whatever reason they go away and the capital is there, well, you should have priced the lease accordingly.
Just like someone renting out some property, all of a sudden the market moves.
Well, okay, well maybe you shouldn't have priced the lease at 10% and fill a market.

Speaker 3: 00:47:14

So Yeah, so there is, okay, you could change the whole protocol and basically go, no, leases are all for three days.
And you just pay more.
So there's one model where you just pay the massive chunk up front for the worst case, and then maybe you get refunded or something on the way down, right?
Or there's the other case where you always pay three days in advance.
You only ever lock up things for three days.
And if you want to keep the liquidity, you keep paying, right?
That model would work too.
And that model is much friendlier for, you know, it's probably closer to zero fee routing model where people seem to like the idea of like a subscription plan rather than this massive amount they pay up front.
It solves bastion's problem in a way right.

Speaker 0: 00:47:59

Yeah I think it makes a bit more sense.

Speaker 2: 00:48:01

For a second, like if we kind of look to legacy markets and how they work, we don't tend to try to regulate markets until we've witnessed things that are broken.
This feels like something like until we have formalized what people are actually buying and have a good understanding of what ways it's exploited in the real world, like, we may not want to actually, like, because there are ways for users to protect themselves, right?
Like, there are ways that you can develop reputational systems and whatnot in the interim.
And I think that is probably not the best idea to try to like sort of regulate this market before we even understand what it is.

Speaker 3: 00:48:43

Yes and no.
I mean, to some extent you build the minimum viable protocol to enable things and that creates the market, right?
Somebody has to

Speaker 1: 00:48:51

do something.
The market does exist independent of this protocol as well, right?
That's the other thing.
People are doing stuff, right?
Like Bifindex has a thing now, well, they'll open a channel to you for withdrawal, right?
And they charge you like 50 bips or something like that for it.
So it does exist.

Speaker 3: 00:49:06

Yeah.
And this is where T-BATS point about just don't enforce it, right?
Just you rely on market man mumble mumble reputation hand wave something and go, well, we don't actually need a protocol as such for that.
You just need it.
You just need an advertiser advertisement protocol, basically.

Speaker 1: 00:49:23

You can you can decouple the advertisement from the contract.
Unfortunately, one last question for D-Best.
Like, is there a scenario in your mind in which the seller doesn't lose?
Because it feels like you're arguing that in these cases, the seller always loses because for whatever reason, they didn't charge enough.
And it's not clear exactly what they were selling, things like that.
What scenario does the seller not lose for you?
That seems like what you're arguing, if you're enforcing the time lock the seller always loses because the channel can close or they can splash something like that.
Isn't the answer the seller should just price something more properly basically and take into account their own costs and not do a six month lease or something like that?

Speaker 0: 00:50:00

Yeah, but the thing is there's only ever new opportunities to use your liquidity.
So what is already locked is deprecating in a way.
You only ever discover new ways of using your liquidity and not the other way around.

Speaker 1: 00:50:15

I feel like you're viewing it as zero sum because whenever you sell this liquidity, you basically get that upfront payment, right?
So much like mining pool bearish reduction, basically.
So obviously you get the upfront payment and then there's a speculative aspect, which is routing fees, right?
Which aren't guaranteed, but you got a guaranteed payment basically, which actually helps offset the rest of your cost of your own routing on itself.
But then similarly, if someone really needed to pay for that channel, they're probably going to be getting some routing fees towards that.
So now you're basically able to de-risk your future routing fee expenditure by getting paid up front basically.
And then also, there's maybe increased likelihood of other fees coming across that channel.
So I don't agree that they're somehow stuck or they're being pigeonholed.
Well, they're the one that entered this contract and they should only do so if they think it's economically advantageous.

Speaker 0: 00:50:58

The issue is that that's only the case if you can guarantee that the amount that is locked is only what they paid for or less as soon as they start using it.
But what we found on the thread is that it becomes...
No, because if you get a lot more, if you get an unbounded amount more that can be locked, then you're going to get screwed.
But the issue is that getting the solution that we-

Speaker 2: 00:51:18

Splices are interactive.

Speaker 0: 00:51:20

Yeah, but then you're losing out the revenue because if you don't accept that splice, but that was an NS user, they would have generated revenue for you by doing that splice and sending those funds out.
So you're losing out by these.

Speaker 1: 00:51:33

But the thing is, splices aren't the only way to shift around funds and then a channel as well in order to receive more funds.
Right.
So I think we have like, we're going from different bases of the model.

Speaker 0: 00:51:44

But

Speaker 3: 00:51:44

I think, yeah, the user originally wanted in-pround liquidity.
They're an honest user, right?
They actually wanted it, right?
And now they're changing.
No, no, no.
I want to splice in a huge amount because suddenly I want outgoing liquidity.
That's weird.
More likely, they want you to splice more in.

Speaker 0: 00:51:58

We see that all the time.
We see that behavior.
All the people are right now buying in BAM liquidity on Phoenix and then doing splices in.
They're doing it all the time.

Speaker 1: 00:52:07

But like, I guess what's so bad about that?
Where are

Speaker 3: 00:52:09

they getting the funds?
I don't...

Speaker 0: 00:52:11

Because some of them are doing some DCR and cashing in, splicing in regularly from I don't know what regular payouts.
Some of them just decide that, oh yeah, they have some on-chain funds and they want to be able to make payments.
So they splice in, which makes sense.
They want to make payments.
People want to make payments and receive payments.
So at some point they'll want to be able to make more payments without having received.
So they will split funds in.

Speaker 1: 00:52:34

Like in your case, if you don't care about the one channel to rule them all, they can open another channel with those funds, right?
And they're paying the exact same change.

Speaker 0: 00:52:42

Because a

Speaker 1: 00:52:42

lot of things won't

Speaker 0: 00:52:43

work anymore if you do multiple channels.
For example, you can't guarantee the safety of Zeroconf swap in Potentium if you do multiple channels.

Speaker 1: 00:52:50

Yeah.
I mean, yes.
I think there's a bunch of additional inputs right now that I guess are factoring into this, which I think are product decisions on your behalf, but they don't necessarily exist universally.
And that you can just open that other channel, they're going to pay the exact same fees.
That lets you partition it at the output level, basically.
But it seems like we're leaning towards decouple advertising from enforcement.
Advertising is one thing, people forgot enforcement and contacts because we need more.
We got to get more information on that for the market.

Speaker 0: 00:53:17

Yeah, and see how complex it is, because once you start doing these things with multiple outputs, it really becomes complex.
And if I can avoid writing that code, I'd like it.
All right, so let's do something else before we end.
I know that Val wanted to talk about async payments a bit more, if she's still around.

Speaker 2: 00:53:41

Oh, yeah.
Not sure if it's the time to start right now, but I've been working with Matt and Jeff recently on a new scheme for adapting Bolt 12 for async payments.
And I've been working on a gist for that, and I'm hoping to send it out to the Bolt 12 Discord soon.
So that's kind of my update.
But yeah, it is kind of weird because we don't want the offer size to blow up, but we want the sender to be able to fetch an invoice from the receiver's LSP.
So there's definitely going to be some stuff for you to review soon.

Speaker 0: 00:54:15

Perfect.
All right, then we're at the one hour limit.
Is there something someone wants to talk about before we end?

Speaker 1: 00:54:27

Just other stuff.
So we have an implementation of STFU now, part of the network equipment stuff, and merge.
We might have an intern coming on soon.
I think they'll start to work on the peer backup thing.
There's like a small cut out of like a my first spec type of thing, nothing too super elaborate.
We're continuing to push out on one of the paths, trying to get the routing support into .18, which hopefully should just let people just do more wide-scale testing, which is something I think would be super good just to get better feel on UX and things like that.

Speaker 0: 00:55:00

Cool.
Cool.

Speaker 4: 00:55:01

I have a question.
I've been doing a bunch of cross splice testing stuff, and I realized that the zero set output for the funding channel makes the cross splicing quite hard, because they're obviously invalid for the other splices.
I was wondering if anybody remembered like why those are zero?
Was there an important reason?
Or can we just change the spec to make them not zeros when anyone remembers kind of thing?

Speaker 0: 00:55:28

What do you mean?
Which zero output?

Speaker 4: 00:55:31

The channel funding output ends up being 0 to start, and then it gets swapped out to the actual amount later on.

Speaker 0: 00:55:39

I don't see what you mean.
I don't have anything that looks like that in our spacing code.

Speaker 4: 00:55:46

Oh, Well, maybe we can just drop it then.
I believe it is in the spec.
But I was wondering if anyone remembered why or what the story was there.

Speaker 3: 00:55:56

I think the idea was to have you have an output that's going to be the channel funding.
But you don't know what the amount is yet until everything's happened.
So you've got to mark it in some way as being an unknown.
And I think we put out a zero output for that and then we later on replace it.
That does not work if you're doing...
The multi-channels cross-splice, which is a case we definitely want to get to right that the other side gets upset they see the zero output but I can't remember if it was actually in the spec or just the implementation detail

Speaker 0: 00:56:33

Yeah that only looks like an implementation detail because you know everything beforehand when you send a splicing it and splice ACK you have a contribution so you have the amount of that funding output.
It is fixed before you send your first TX at input.

Speaker 4: 00:56:49

I think my memory of the spec is possible I did it wrong.
Was that we actually send over the output with zero amount in it.
But I mean I'm guessing that like we could just kind of get rid of that but I just wanted to be really certain.
I think the other way we can detect channel funding output is just look that the script matches correctly.
And then there's a question of what if somebody maliciously adds an output that looks like the funding output.
Would that matter?
And I can't

Speaker 0: 00:57:17

think of an

Speaker 4: 00:57:17

answer here I would.

Speaker 0: 00:57:19

But then the way we do it and when I last checked the spec, the spec seems to do that.
You send a splice init, splice hack.
Both of these contain your contribution.
So from your contributions and the current channel output size, you know exactly the size of the next channel output.
So you know that.
And it's the one who initiated Splice who has to send TX as input for the command input.
And it has to be a new format that we haven't defined where you don't have to send the previous TX because both sides have it because it's the current funding transaction.
But apart from that, it should be quite simple.
I don't see the need for zero amount output and I don't see the need for watching if the other guy had the same input because it would just be fall under the same rule that two people try to add the same out point to a channel so it's just rejected.
But yeah we can keep discussing it offline to make sure we figure it out.

Speaker 4: 00:58:24

Okay, sweet.
Yeah, I got the sense that it's not needed, so it sounds like

Speaker 1: 00:58:30

it's probably the case.
Yeah.

Speaker 0: 00:58:33

Yeah, I think it's not needed.

Speaker 3: 00:58:35

I think we see a lot more of this where we're basically splicing out of one channel straight into another as a common case of rebalancing, particularly for things like, obviously the Eclair node may wanna do this mega splice where you basically rebalance 12 channels at once kind of thing.
And that I think is going to be a common case, right?
In an all lighting world, everything's a splice, right?
So cool.

Speaker 4: 00:58:57

Well, Tivok, do you want to do an interop soon?
What's your splicing work schedule like?

Speaker 0: 00:59:04

Yeah, I think we can start working on that, because I think we don't use the same exact TLVs because they are not fully...
Some are missing from the spec, I think, at least for the reconnect part, for example.
But then an interop will show us exactly where it breaks, and then we can iterate on the spec.
But I was hoping we could merge the dual funding PR so that we could clean up the quiescence and then splicing PRs where it would make it easier to review and easier to iterate on.
But since we still have this previous TX thing to finalize on the dual funding one, it's going to be delayed a few weeks or hopefully only a few weeks.
But then after that we can start doing more splicing stuff.
Okay, cool.
Okay, cool.
All right, then let's end it now.
Thank you everyone and have a good week.

Speaker 1: 00:59:59

Cool.
Thanks.

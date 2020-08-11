layout: true
class: typo, typo-selection
count: false

---

# Architecture Kata

>   "How do we get great designers?
>  Great designers design,
>  of course."


>  "So how are we supposed to get great architects, if
>  they only get the chance  to architect fewer than
>  a half-dozen times in their career?"



???

I want to talk today about how to design things. It so happens that this is going to seem really familiar, because it's explicitly what I do all day every day with you lot. One of the challenges of becoming an architect is that you only get to practice architectural skills when you're designing systems, and that's not an opportunity that you get very often as an engineer. Architecture Kata are a way of practicing those skills without you actually running amok and rewriting everything as event-sourced actror systems in scala on top of kafka on a kubernetes cluster in GCP.

---

# At the start of a project there's three questions to answer

???

A lot of what I do is answer these three questions. The way it works is this: Jon, the CTO, will ping me in slack and he'll say "hiya" and I say "hi Jonb" and he says "just a crazy idea" and i say "uh oh" and he says "what if we bought a major national car dealership and they had a php monolith and it was actually good. What would we do then? Would we use it? Could we integrate with that in a sane way? How would that actually work? I assume we wouldn't want to fire all our engineers and hire a load of PHP devs" and I say "hahaha this is a hypothetical situation I'm making up for a talk, isn't it, Jon?" and he says "hahahah, yes of of course, this would never happen in real life."

--

* Could we?
* Should we?
* How would we?

???

These are the three questions we have to answer at the start of a project. Could we do this thing? Is it possible given the constraints we have, about deadlines and skill set? For example, the data team would like it if we could identify all traffic that came from Cazoo employees or our integration tests. It turns out that doing this, when everyone is remote, and uses multiple browsers on their personal phones and laptops and whatnot is prohibitively difficult and intrusive, so the answer is "No, no we could not". Should we do this thing? Sometimes people get excited about a piece of technology and they're like "hiya bob, I think Rust is super cool and we should use it for all our systems, and also Kafka" and I'm like "okay, go write me an RFC on how to make that happen, and how you're going to train people to use these tools, and how we're going to use them alongside our existing tools" and it turns out that, actually, we should not _not_ use Rust for any of our systems, with the plausible exception of a trial run on Cloudflare workers, because that would be a terrible and disruptive change with little practical upside.

---

# How would we build this thing?

???

Which leaves us with the third question: assuming that it is both possible and desirable for us to do this thing, how on Earth would we do it? This is the exciting bit of my job, because it's where I get to actually think about technology and how it works.

---

# I am not an original thinker on this

https://thepracticaldeveloper.com/practical-software-architecture/

???

I learned all of this stuff from a guy called Ian Cooper, who has a terrifying beard, and I was lucky enough to work with him for about 5 years at Huddle. He taught me a hige amount of stuff about TDD practice, and how to organiser a codebase so it doesn't turn into goo over time, but he also had a "process" in inverted commas for designing stuff that he never really explained except to say "it's part of the Crystal method" whatever the hell that is. But I sort of learned it by osmosis.

Thankfully, since them, some smart people have written about architecture in ways that are accessible. I always send people to this link, which is a great distillation of architecture practice, so you can just go read that and understand what an architect does, instead of having to work with me every day for 5 years and hope that I rub off on you.

---

# TL;DR

* Understand the vision
* Document the constraints
* Decide architectural principles
* Agree the quality attributes
* Author proposals

???

If you're too lazy to even read that document, then that's okay, friends, I've got you. These are the five steps in designing a system. Do these five things, and do them well, and you too can be a chief architect at a fancy tech unicorn. This is what we're practicing when we do an architecture kata - these five steps - and that's what we're going to do today. We're going to take a _problem_ and we're going to design solutions to that problem by working through these five steps.

---

# Understand the vision

* How will the organisation be different when we've done this project?
* What will be the immediate impact?
* How will we change the solution over the following year?
* What will we achieve in five years?

???

Step one is to understand what the hell we actually want to do. What is the thing that business wants to happen? For example: we want to build an application that tracks the location of a vehicle. We want this so that we know where all our cars are. Not knowing where our cars are is a big problem for our business process. Over time we want to extend this solution to understand where the car _should go next_ in its journey through our processes. This is the basis of our fancy vehicle inventory management system that will completely automate the tracking of vehicles across all our refurb and retail sites, without manual data entry, automatically creating jobs for workers to get a vehicle ready for sale. That's a vision statement. It tells us what we want to do, and why.

---

# Document the constraints

* Time
* Money
* Knowledge
* Tools

???

Step two is to say "that's all very well, but in the real world, what will stop us doing that?" Well, we need to deliver _something_ in the next 6 months. The only team who are available to work on it have no front-end expertise, we're currently in a hiring freeze and won't authorise the purchase of new software, the people who will use the system are non-technical and work in a busy, oily, noisy environment with poor wifi, and so on.

---

# Decide architectural principles

https://www.notion.so/teamcazoo/Architecture-Principles-c8cfb524f3fe40148794163f1360b9c0

???

Ken had a great definition of principles recently: principles are rules of thumb that you can use when you can't choose between two options. We should be able to go back to our principles and use those to make a choice. I did this on the ImperialNet project, and it's turned out to be really helpful, but I don't tend to do this on smaller projects, and we'll skip it for today. The _reason_ I don't usually do this on small projects is that the architectural principles tend to be the same for _all_ of our systems: we build event driven, serverless microservices.
Our architectural principles are a little bit documented at that URI, but you can find them in notion by searching for architecture principles.

---

# Agree the quality attributes

???

We know what the functionality is: it has to track vehicles in a workshop environment without paper, but we also ahve to think about _non-functional_ requirements. How fast does it have to be? What happens if it goes down? What happens if a bad person infiltrates and steals our data? What if they can _add_ data? How will people use it with gloves on? How can we test this piece of software?

We use a handy mnemonic for these: MUSTAPO which stands for


--

* Modifiability
* Usability
* Security
* Testability
* Availability
* Performance
* Observability

???

Modifiability is the ability to change or reconfigure the system. Some systems don't change at all once they're deployed, others need to be constantly tweaked by end users to change the behaviour. Usability is our ability to provide a service to users without pissing them off. This isn't just our end users, we should think about the developer experience gere, too. If it's a nightmare to work with the codebase, that's a usability flaw.
SEcurity is our ability to privide a service in the presence of hostile actors. What happens when this system is comprimised, is that just annoying, or is it the end of the business?
Testability is our ability to test the system without, yunno,actually buying a car on finance and sending it to manchester. How can we design and segregate our code so that we can gain confidence in it easily and assert its behaviour?
Availability is the abiilty to provide a service consistently: what happens when this system goes down? WIll we immediately lose mnoney, or is it okay for us to take things offline for 6 hours overnight?
Performance: How fast does this have to be, and how many users does it need to serve? What happens if we miss those targets? For a real-time stock market bidding tool, performance is critical.  For an overnight batc h CSV processor, it probably isn't.
Observavbility: How can we ask questions about this system when it's in production? How will we know if any of the other attributes are being violated?

---

# Author Proposals

???

Then you go and design a thing, you have some crazy ideas, based on the things that you have built before, or that you read about on Hacker News, and you test it against the quality attributes and constraints and so on. When you've got something that seems to work, you go talk to your team and say "hiya, I have this crazy idea" and you hope they don't laugh.
It's helpful if you make some diagrams, because people like diagrams.

That's it! 

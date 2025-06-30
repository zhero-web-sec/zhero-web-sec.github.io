---
title: "Bug bounty, feedback, strategy and alchemy"
collection: portfolio
permalink: /thoughts/bugbounty-feedback-strategy-and-alchemy
date: 2025-06-30
---


Honey attracts bees, and like many others who occasionally share moments of success, I often get asked recurring questions about bug bounty hunting: how I got started, what advice I’d give, what roadmap to follow, and so on. I figured it might be worthwhile to put some of my thoughts, experiences, and perspectives into writing for anyone curious about the subject.

<img src="/images/bb-alchemy-1.png" style="display: block; margin: 0 auto;">

A very brief autobiographical note is necessary to lend some legitimacy to the following remarks: with no background in IT, I've been practicing bug bounty for almost three years, and I started learning web-based offensive cybersecurity around the same time. I generate six-figure revenues per year through bug bounty hunting, across all platforms, without too much difficulty, and I've been making a full-time living from it for a while now.

That being said, what follows is sometimes subjective, stemming from my reality and experience. Furthermore, everything that follows is from the perspective of a web-focused bug hunter and is intended for bug hunters.

<h1>Bug Bounty Hunting: the autodidact’s battlefield</h1>

**On paper**, bug bounty is a compelling concept, it provides a legal framework and is built on a well-thought-out economic model.
Calling it pure meritocracy isn’t entirely accurate, since by definition, you can work for free if you find nothing.
However, who you are, what degrees you hold, who you know, or where you live doesn’t matter.
No interviews, no schedules, no race to meet arbitrary standards.
There are rules, rewards, a defined scope and the game begins.
You discover a vulnerability within the rules? You earn a reward.
No ceilings, no safety nets, **only skill reign**.

**In practice**, it’s a bit different: humans are still in the loop.
And humans are imperfect, sometimes biased, dishonest, or simply incompetent.
At its core, it’s a business, not just a playground for passionate hackers (*or unilaterally*), and that comes with its share of realities.
This inevitably leads to frustration, disappointment, and at times, a strong sense of injustice.

That being said, the positives far outweigh the negatives in my opinion. It opens many doors and offers you a range of possibilities. I continue to find the concept fascinating after all this time, it's literally a game whose consequences have an impact on real life, including companies, users, or even your bank account. You have to be a learner, and continually improve your skills in what you think is most relevant at time T. I'm not old, but not particularly young, and I've personally learned a lot *in general* from my experience in this environment. It's a kind of accelerated R&D process that allows you to quickly verify the accuracy of your choices. A good school for self-teaching, opening the appetite for curiosity and knowledge.

<h1>Banalities: methods linked to objectives</h1>

As I progressed, I quickly realized that bug hunting in the traditional web ecosystem takes many different forms. These forms/methodologies are often not comparable, and their relevance ultimately depends on each individual’s skills and objectives:

- make it your main source of income
- make ends meet
- reputation (*in the literal sense of the word, at least I hope*), glory, fame, aura farming
- springboard to enter the industry / find a job
- practice, train, learn, perform
- to have fun

Most of these objectives are legitimate, and the notion of "success" obviously depends on the goals that have been set. I see many talented people who fail at bug hunting, and one of the reasons is that their methodology or approach doesn’t align with their primary objective.

The choice of objective must be in line with reality and this includes several variables such as time, current knowledge, cost of living in one’s country, experience, or learning capacity. And even if one of these factors is disregarded, it should be done with full awareness of the potential consequences.

The same applies to the roadmap: two people learning with the same goal, let’s say, making it their main source of income, will not necessarily follow the same path depending on their circumstances. A young man without responsibilities, living with his parents, might take the long road, laying the foundation brick by brick. Meanwhile, someone undergoing a career transition, with limited time and financial obligations, continuing to study in the evenings after work, may take shortcuts and head into the battlefield more quickly, much less equipped (equipped nonetheless), then will learn as he goes. Time is limited, and being a purist in your approach doesn't lengthen it.

There are a lot of assumptions about bug bounty in general, one of them being that it’s not something you should pursue with the goal of making a living. Web security isn’t inherently complex, which makes (*web-focused*) bug bounty accessible to almost anyone. But that low barrier to entry comes at a price: intense competition and, at times, a particular social status within the cybersecurity community. That said, accessibility is one thing, anyone can find a vulnerability, but turning it into a sustainable income is something else entirely. The entry level is no longer the same, and it’s not for everyone.

Ideally, you should take things gradually and avoid letting go of one branch before grabbing the next. I’ll assume I’m speaking to responsible adults with their heads on straight, so I’ll skip the obvious. Personally, I quit my relatively well-paid, non-IT job in France when I was making $0 a month, with responsibilities on my back and only a few months of runway.

Would I advise it? No, but I wouldn't advise anyone against it either, being too wise limits losses as much as gains. Your choices and your investments are up to you my dear friend, **just be lucid**.

<h1>Bug hunter profiles - Select your character</h1>

So there are several ways to practice bug hunting, translating into various forms or methodologies, which are more or less relevant depending on the objectives and the hunter’s level. If the goal is strictly to make money in order to fill your fridge and pay your bills, it can be naive - depending on your skill set - to simply look for bugs in a generic way, or to focus only on vulnerabilities/programs you like, regardless of certain statistics. It’s now a kind of business for you, and like any business, you need some semblance of a business model or strategy. Your whims, passions, and preferences are no longer necessarily relevant unless they align with a winning horse. If that’s the case, all the better, it’s well known that we’re more efficient when we’re having fun.

<img src="/images/bb-alchemy-2.png" style="display: block; margin: 0 auto;">

Let’s personify certain hunting styles at a very high level, and examine each one’s relevance - for the sake of our example - in the context of full-time bug bounty practice (*in a country with an average cost of living*), or at the very least, for earning money consistently, assuming in each case that the hunter has a solid skill level (*very abstract, I grant you*):

- **The main app guy**: Focuses on core apps, one at a time, digs deep into all features, monitors updates, and knows almost all the developers’ names (no). Pure hacking, very enjoyable, but requires a level well above the rest to realistically hope to make money on a regular basis this way, the competition being very strong. Duplicate rate: high/low, depends.
- **The recon guy**: Avoids core applications and looks for programs with wild scopes, good for stretching the attack surface to the maximum, monitors changes, and looks for vulnerabilities in exotic subdomains. Unearthing assets the company didn't even know existed. Can make some money, but there's a big loss of revenue without automation of these skills. Duplicate rate: medium.
- **The master automator**: Automates everything possible. It’s usually a cross between a recon guy and another class. Invests hundreds/thousands of dollars in their infrastructure, this is the industrialization of bug bounty hunting, and generally the ones occupying the top spots on the leaderboards. The latter, having large in-scope asset databases, do not hesitate to get in touch with zero-day researchers, for exploitation on a large scale through win/win collaboration. At the crossroads between several skill sets, it’s one of the most relevant and profitable ways to make money and climb the leaderboards. Duplicate rate: these are often the authors of the original reports on which certain classes take dupes.
- **The low-hanging fruit eater**: Only looks for low-severity vulnerabilities without complexity, easily identifiable, not always accepted/rewarded by programs (*sometimes OOS*). Does not linger on an asset, and quickly jumps to the next one once the first checks are done. Allows for quantity at the expense of quality. Can generate some money fairly regularly, but reputation, impact, and signal are often the price to pay. He hates *the master automator* for obvious reasons. Duplicate rate: high.
- **The architect**: Pure hacking, pure art. Patient, able to spend several weeks on a vector to reach maximum criticality. Chain several vulnerabilities together until reaching the desired impact and fairly regularly obtain five-figure bounties. Elite and rare class. Can make a relatively easy living from it. Since the frequency of bugs is lower than others, some financial education is naturally necessary in order to manage times when there is no money coming in. Never reports low vulnerabilities and holds a grudge against *The Low-Hanging Fruit Eater*, the latter burning certain cards necessary for his exploits. Duplicate rate: possible on certain bugs necessary for his chains.
- **The zero-day researcher**: Spends time reading code and debugging, looking for vulnerabilities in software widely used by bug bounty programs. After a responsible disclosure to the relevant teams, reports the vulnerability separately to the programs with impacted assets. Can generate money frequently depending on the recurrence and the software involved. Links the artisanal side of research with the industrial side of spraying at scale. The effect can be multiplied tenfold if coupled with the right classes. Relevant practice, quite different from the preceding categories. Sometimes in contact with the *master automator*. Duplicate rate: null, by definition.
- **The niche specialist**: Specializes in one type (or family) of vulnerabilities and masters it better than x% of other hunters. Finds what others miss and rises above the competition. Its relevance depends on the choice of specialization, which I’ll come back to a little later. If the choices are good, it can obviously make money frequently. A school that [I have recommended in the past](https://x.com/zhero___/status/1848069476463301045), and that I continue to recommend. Duplicate rate: low
- **The opportunist**: He’s average across the board. Keeps an eye on market fluctuations, tracks the latest exploits, and acts fast when opportunities arise. Generally good at making money, with or without bug bounty. 
- **The AI ​​bot**: Scares some of the above classes, and haunts the nights of *The low-hanging fruit eater*. Will probably erase those who do not adapt and/or adopt it. 

Let's stop here; the list isn't exhaustive, and as you've guessed, I sometimes exaggerate to highlight certain trends. Things are obviously not always that simplistic, and hunters are often a mix of several of these (sub)classes. Each of these classes contains multiple levels of depth in terms of skills, and results will primarily depend on that.

Each methodology has different entry levels to generate money frequently/live off full-time, and they are not necessarily accessible to a beginner/average hunter, for whom it will quickly become difficult to stand out from the competition, and who risks an overdose of duplicates on the easiest paths. Being average at everything, without excelling at anything is impractical, brings a lot of frustration, and if you add to that the scams or questionable behavior of certain programs on the few valid reports that slip through the cracks, it becomes the perfect recipe to make some give up.

<h1>Not an elite generalist? Life is short, become a niche specialist and see later</h1>

Always under the prism of someone wanting to make money, and ideally practice this part/full-time, it's important to get out of an overly academic mindset. You don’t need anyone’s permission or validation for your way of proceeding (*in terms of learning processes and methodologies*), and being too intellectually rigid will hold you back on many levels, starting with your creativity. Burn some stages if they’re not hierarchically dependent. Tackle things bigger than yourself, no one cares. You’ll quickly realize if you have gaps, and you’ll fill them along the way. As a hunter, only the results count, just choose your weapons and your targets thoughtfully and coherently with regard to the market.

Speaking more concretely about weapons, choosing a niche is a very good way to avoid competition and can be a vector for "success". [Everything is accessible/learnable today](https://x.com/zhero___/status/1931166243656241512), the only currency is your time. And based on that, building your skill set inherently involves risk when the primary goal is the alchemical process of turning knowledge into gold: Is skill set X worth Y of your lifetime?

You have to see this for what it is: an investment, and the capital is your time. This is where some talented people fail (*and by failing, I'm referring to the initial chosen goal, money in our example*): technically brilliant, but bad at investing. Specializing in an obsolete, outdated technology just because you're comfortable or have an history with it is obviously a mistake. Specializing in a highly technical field that isn't really exploitable on a large scale will surely allow you to sometimes achieve some nice exploits, but not to regularly make gold from it. The same goes if you choose your specialization solely based on your preferences or appetites.

<img src="/images/bb-alchemy-5.png" style="display: block; margin: 0 auto;">

Don't expect a precise answer. An investment is personal, and you alone must bear the responsibility. After all, very long hours in front of your screen are at stake. However, here are a few important points to consider:

- You need to determine whether this is a long or short-term investment, in order to define how much time you're willing to invest before seeing your first potential results.
- Take into account the current market situation and possible future technological shifts, including the proliferation of AI bots in an era increasingly driven by acceleration.
- Move toward what others avoid. People are often lazy and avoid complexity, or what merely appears to be complex. If all other lights are green, embrace it.
- What technology/component is often present but rarely/less discussed, exploited, or explored, while being almost systematically in-scope? A quick search on X (ex-twitter), or a Google dork can be a good indicator to validate an idea.
- If it can be automated at scale, so much the better, but it's obviously not a prerequisite.
- Don’t hesitate to read research papers on novel attacks (new or old); you might find a gold mine hidden under the dust.

<h2>Decision made? Hit the road</h2>

I won’t list all the resources for learning and training, to avoid reinventing the wheel yet again. Regarding the method, however: after the fundamentals, I personally like to start with a broad picture of what I’m aiming for. I create a mind map based on a high-level understanding of the subject, then gradually approach each component separately, placing them on the map as I go. This is much more effective than the opposite: learning chapter by chapter and "unlocking" the map with only a vague idea of where you’re headed. It’s a good practice to forbid yourself from learning something without knowing its concrete practical use (*of course, when possible*). The more control you have, the more things make sense, and the more they make sense, the more effective you’ll be.

<img src="/images/bb-alchemy-3.png" style="display: block; margin: 0 auto">

There’s no point in focusing on pure memorization when taking notes, only understanding matters. Prioritize diagrams, charts, or sketches. You won’t be hunting without an internet connection, and therefore not without an AI bot to - at the very least - refresh your memory when needed.

Once the basics are covered, both in theory and practice, it’s time to move on to research papers, RFCs, various stacks, and low-level concepts whenever relevant. Keep digging deeper until you make a difference.

<h2>Transforming your knowledge into gold</h2>

You’ve mastered your subject, finally earned your first bounties, and you're no longer too worried about potential duplicates when submitting reports: congratulations, you’re now a niche specialist (*at least relatively*). At this stage, the best move is either to acquire a new class or to collaborate in order to maximize your earnings.

The first option is the strongest in the long run, and you have two solid paths forward:

- Become a *master automator*, combining deep recon, automation, and your specialty to automate your secret sauces at scale (*if applicable*)
- Become a *zero-day researcher*, looking for vulnerabilities specific to your niche on widely used software, to finally mass report on bug bounty programs with impacted assets (once the usual procedures have been carried out)

<img src="/images/bb-alchemy-4.png" style="display: block; margin: 0 auto; width: 75%">

Each of these classes has its own roadmap and its own cost in terms of investment, and therefore time. Best case scenario, you're starting to get really comfortable, and things are going pretty well for you. In the worst, you made a bad investment and lost your stake. Depending on your reality it will be more or less regrettable, but that's the game, the happy ending is not guaranteed. You have still gained knowledge even if it is not directly usable as such and that it does not serve your purpose in the current context. Your learning capacity has surely leveled up, and if you're a player, it's time to start the process from the beginning and invest again.

<h1>Great souls, great ambitions</h1>

If you’re generating enough income that you don’t have to worry about the next X months, don’t fall into the trap of comfort. The world moves fast, things evolve quickly, and what works today won’t necessarily work tomorrow. It’s time to allocate some of your time to explore another niche, deepen your current one, or even learn something entirely different. It might feel counterintuitive and you can easily feel like you’re "losing money" by learning something new, even though you’ve already found a winning formula. Manage your time budget strategically, structure your day, week, or month in a way that lets you keep learning while still working. Always keep the long term in mind, it’s better to earn less today than to earn nothing at all tomorrow.

Those X months of financial buffer are a valuable asset that you can invest to throw yourself body and soul into learning anything as long as it aligns with your goals. It's a luxury that few people with responsibilities can afford. Additionally, it may be wise to diversify your sources of income and start building your financial literacy.

Money has been discussed a lot here, nothing unusual from the perspective of a full-time independent bug bounty hunter. But it's time to aim higher, to seek to move things forward in your field, and to do your part. I find it fascinating that much of the technology we use today is the result of a vast stack of universal knowledge, built over centuries by countless contributors. Narrowing it down, you've probably reached this level thanks to numerous research papers, courses, and videos shared for free by people around the world. Aren’t you, in some way, indebted to that stack? Now it’s your turn to contribute with research papers, novel findings, presentations, useful tools or simply good advice. If it worked for you, it might work for someone else too.

Thank you for reading.

Al hamduliLlah, the praise belongs exclusively to Him.

[zhero;](https://x.com/zhero___)

*Published in June 2025*
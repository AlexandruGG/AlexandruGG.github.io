## Reviewing Code Made Me a Better Developer

Most people working in software development are aware that significantly more time is spent reading code than actually writing it. It's simply a necessity, whether to understand an existing system, extend a feature set, or learn how to work with a language or technology we're not too familiar with.

One instance of this that always stood out to me was reviewing other developers' code. Whether you call them _pull requests_, _changelists_, _merge requests_, _code reviews_, or _patches_, it's undeniable that they are an important part of the software development process.

In this article, I want to share my thoughts when reviewing code, and the elements I believe made me a better reviewer. I was inspired to write it due to a Hacker News [post](https://news.ycombinator.com/item?id=31047409) advocating for interviews in which developers are tasked with [reading code](https://freakingrectangle.com/2022/04/15/how-to-freaking-hire-great-developers/) and discussing it, rather than having to complete coding exercises.

---

### Personal Context

#### Early Days

My first role in a software engineering team was that of a _Test Automation Engineer_ - I was tasked with developing end-to-end tests for the company's web app. As this was not meant to be a permanent role in the team, I started getting more involved in the software development process and helping out on the front-end side with components and small features.

I believe that one of the most important things I started doing is looking at every new pull request containing front-end code. This not only allowed me to understand the business and the direction it was moving towards but also helped me learn a lot as a junior.

While I wasn't knowledgeable or confident enough to leave many comments, I tried to not shy away from asking clarifying questions whenever I wasn't sure _why_ the code was written in a certain way.

Eventually, I became confident enough to deliver front-end features on my own, and, being the curious type, started getting involved on the back-end. Again, I applied the same approach - reviewing PRs and asking questions.

#### More Power = More Responsibility

Over time, I developed a reputation for being a thorough code reviewer. The company I was working for at the time, [Goodlord](https://github.com/ohgoodlord), introduced a system of [code ownership](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners), and I was given the responsibility of acting as one of the code owners for the back-end code (mostly PHP and Scala).

The code ownership system went hand-in-hand with a _2 PR reviews requirement_ - each PR had to be reviewed by at least one code owner and another person. While this system has its pros and cons, it worked for our team at the time; the expectation was that, in time, other developers and new joiners would also become code owners.

My approach with PRs evolved - while initially I was using them as more of a learning tool, now I was responsible for upholding the team's coding standards. Equally importantly, I felt I had to use code reviews as a teaching opportunity where applicable - having known how much I benefitted from them in the past.

### Tips & Tricks

#### My Approach

There is a myriad of things to check during code review - I will try to distill the important aspects of my approach.

Should go without saying, but the zeroth step should be to agree as a team on what's required when submitting new PRs:

- tests to be included - unit, mutation, integration, end-to-end
- code standards & style to follow
- documentation to include

All of this _should_ be enforced by build tools automatically. Don't spend precious time _in code review_ arguing about what test coverage is required or which line the curly brackets (`{}`) should be on.

With that out of the way, here's my suggested approach:

1. if the PR is for a product feature, check the ticket first (hopefully there is one?) - try to understand what the feature is and how it's supposed to work

2. read the PR description and any comments left by the author

3. look at the file structure and assess whether it makes sense in the context of your project

4. review the code (and the tests!), leaving comments or questions where needed

That's it? Well, yes, it's not "rocket science". But here are a few more tips:

- if your team is not already using one, consider introducing a [mutation testing](https://en.wikipedia.org/wiki/Mutation_testing) tool. In my experience, tests are not as thoroughly reviewed as the rest of the code, and a mutation testing [tool](https://github.com/theofidry/awesome-mutation-testing) can help improve test quality significantly; stay tuned for a future article on it!

- assess whether the PR is _too big_ - this can depend a lot on the codebase and the team's working style, but a good rule of thumb for me is to see a PR as "too big" if it's doing too many unrelated things or would take more than 20 minutes to review. Large PRs are harder to review - don't be reluctant to ask the author if changes could be split into separate PRs

- be timely - when a new PR is opened and particularly if the author specifically asked for your review, don't make them wait. Again, this will depend on the team's pace of work, but typically if I'm not able to review the code within 2-3 hours I will let them know; this way, at least they are aware and have the option of waiting for my review or requesting someone else's

- try to offer code suggestions, examples, and further reading where applicable - especially important with more junior developers; reading the same suggestions and best practices from multiple sources will give a person more confidence that the work they are doing is of good quality. One of my favourite resources to share was [clean-code-php](https://github.com/jupeter/clean-code-php)

- if you find yourself leaving the same comments on PRs from different authors, consider implementing a custom rule using your static analysis tool of choice

- don't rush - set aside 15-20 minutes where you can fully focus on the code review task

- checkout the branch if needed - this goes hand-in-hand with the previous tip; seeing code in an IDE can make it much easier to understand. Luckily, since the launch of [github.dev](https://github.dev/github/dev) this has been made much easier to do directly in the browser

- be courteous and respectful when writing comments - something that seems obvious to you might not be to someone else. I find that posing questions and asking for the author's _opinion_ on a suggested change can go a long way vs simply commenting something like "I think this should be done like so"

Lastly, a resource I found invaluable is [Google's Code Review Developer Guide](https://google.github.io/eng-practices/review/). While the document's purpose is to describe the company's code review processes and policies, it provides detailed guides on how to both review code and submit code for review as an author.

#### Happy Reviewing!

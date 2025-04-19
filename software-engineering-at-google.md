[Back](/)

# Software Engineering at Google: Lessons Learned from Programming Over Time
Titus Winters, Tom Mashreck, Hyrum Wright (2020)

---

## TL;DR
- All exposed behaviors _will_ be used (Hyrum's Law). Keep things hidden by default
- If you’re not failing, you’re not innovating
- It pays off to make a small effort to understand peoples’ personalities and working styles
- Consistency makes it easier to understand code, modularize and scale it, for tooling to work with it; it facilitates mobility across teams
- Consider restricting unusual or tricky language constructs (e.g., reflection)
- Treat documentation like code, with `CODEOWNERS`; identify: who, why, what, when where
- Organize tests by speed
  - Fast tests can't sleep or make blocking calls
  - Medium tests can't make network calls
  - Slow tests are mostly smoke tests
- Tests should be hermetic (contain all the setup)
- “If you liked it, then you shoulda put a test on it.” (Beyoncé Rule)
- Tests should be DAMP, not DRY

## Notes
### What is software engineering?
Engineering (vs programming) = longer lifespan, more people involved, more complex decisions, more ambiguity

“It’s programming if ‘clever’ is a compliment, but it’s software engineering if ‘clever’ is an accusation.”

> Hyrum’s Law: with a sufficient number of users of an API, it does not matter what you promise in the contract: all observable behaviors of your system will be depended on by somebody.

> Being data driven is a good start, but in reality, most decisions are based on a mix of data, assumption, precedent, and argument. It’s best when objective data makes up the majority of those inputs, but it can rarely be all of them.

### How to work well on teams
- Humility
- Respect
- Trust

> At Google, one of our favorite mottos is that “Failure is an option.” It’s widely recognized that if you’re not failing now and then, you’re not being innovative enough or taking enough risks.

> By the same token, if you do the same thing over and over and keep failing, it’s not failure, it’s incompetence.

> A small investment in understanding personalities and working styles of yourself and others can go a long way toward improving productivity.

### Knowledge sharing
The Recurse Center’s social rules:

- No well-actuallys: “Well, actually, it’s called GNU/Linux”
- No feigned surprise: “Wait, you’ve never used the command line?”
- No backseat driving: Giving advice without the right context or committing to the discussion
- No subtle -isms: “Even my mum can do that”

> Seek out and understand context, especially for decisions that seem unusual.

### How to lead a team
> If you’ve spent the majority of your career writing code, you typically end a day with something you can point to—whether it’s code, a design document, or a pile of bugs you just closed—and say, “That’s what I did today.” But at the end of a busy day of “management,” you’ll usually find yourself thinking, “I didn’t do a damned thing today.”

> How do you effectively coach a low performer? The best analogy is to imagine that you’re helping a limping person learn to walk again, then jog, then run alongside the rest of the team. It almost always requires temporary micromanagement, but still a whole lot of humility, respect, and trust—particularly respect. Set up a specific time frame (say, two months) and some very specific goals you expect them to achieve in that period. Make the goals small, incremental, and measurable so that there’s an opportunity for lots of small successes. Meet with the team member every week to check on progress, and be sure you set really explicit expectations around each upcoming milestone so that it’s easy to measure success or failure. If the low performer can’t keep up, it will become quite obvious to both of you early in the process. At this point, the person will often acknowledge that things aren’t going well and decide to quit; in other cases, determination will kick in and they’ll “up their game” to meet expectations. Either way, by working directly with the low performer, you’re catalyzing important and necessary changes.

No compliment sandwich, instead lay out the facts:

> We’re quite sure that you’re not aware of this, but the way that you’re interacting with
the team is alienating and angering them, and if you want to be effective, you need to
refine your communication skills, and we’re committed to helping you do that.

Don’t expect things to work out themselves—step in, and early.

> … some are like cacti and need little water but lots of sunshine, others are like African violets and need diffuse light and moist soil, and still others are like tomatoes and will truly excel if you give them a little fertilizer. If you have six kids and give each one the same amount of water, light, and fertilizer, they’ll all get equal treatment, but the odds are good that none of them will get what they actually need.

### Measuring engineering productivity
Goals-signals-metrics framework:

- Goal: desired end result
- Signal: how you know the result has been achieved
- Metric: proxy for a signal: easy to measure

### Style guides and rules
Ask: what goal are we trying to advance?

> There is a nonzero cost in asking all of the engineers in an organization to learn and adapt to any new rule that is set.

> We’d rather the code be tedious to type than difficult to read.

Consistency:

- Experts can glance at some code, zero in on the important parts, and understand what’s going on
- It’s easier to modularize and spot duplication
- It enables scaling, and enables tooling
- It enables mobility across teams and projects

> If conventions already exist, it is usually a good idea for an organization to be consistent with the outside world.

> Our style guides restrict the use of some of the more surprising, unusual, or tricky constructs in the languages that we use.

> This reasoning is behind our Python style guide ruling to avoid using power features
such as reflection.

> We have rules about naming: naming of packages, of classes, of functions, of variables.

### Code review
> In general, engineers are encouraged to approve changes that improve the codebase rather than wait for consensus on a more “perfect” solution. This focus tends to speed up code reviews.

> In general, reviewers should defer to authors on particular approaches and only point out alternatives if the author’s approach is deficient.

> Reviewers should avoid responding to the code review in piecemeal fashion. Few things annoy an author more than getting feedback from a review, addressing it, and then continuing to get unrelated further feedback in the review process.

> Remember that code review is a learning opportunity for both the reviewer and the author. That insight often helps to mitigate any chances for disagreement.

> There is a tendency within the industry, and within individuals, to try to get additional input (and unanimous consent) from a cross-section of engineers. After all, each additional reviewer can add their own particular insight to the code review in question. But we’ve found that this leads to diminishing returns…

### Documentation
> At Google, our most successful efforts have been when documentation is treated like code and incorporated into the traditional engineering workflow, making it easier for engineers to write and maintain simple documents.

> Documents without owners become stale and difficult to maintain.

> Because no process was put in place for adding new documents, duplicate documents and document sets began appearing.

> The way to improve the situation was to move important documentation under the same sort of source control that was being used to track code changes. Documents began to have their own owners, canonical locations within the source tree, and processes for identifying bugs and fixing them; the documentation began to dramatically improve.

> Most engineers are members of a team, and most teams have a “team page” somewhere on their company’s intranet.

Be clear about:

- Who the audience the is
- Why it should be read
- What the purpose is (e.g., a tutorial)
- When it was created, reviewed and updated (consider trigering reminders for reviews)
- Where it should live (e.g., source control)

### Testing overview
> As part of this policy, all new code changes were required to include tests, and those tests would be run continuously. Within a year of instituting this policy, the number of emergency pushes dropped by half. This drop occurred despite the fact that the project was seeing a record number of new changes every quarter. 

> Imagine a hypothetical 100-person team whose engineers are so good that they each write only a single bug a month. Collectively, this group of amazing engineers still produces five new bugs every workday. Worse yet, in a complex system, fixing one bug can often cause another, as engineers adapt to known bugs and code around them. The best teams find ways to turn the collective wisdom of its members into a benefit for the entire team. That is exactly what automated testing does. After an engineer on the team writes a test, it is added to the pool of common resources available to others. Everyone else on the team can now run the test and will benefit when it detects an issue.

> One of the lessons we learned fairly early on is that engineers favored writing larger, system-scale tests, but that these tests were slower, less reliable, and more difficult to debug than smaller tests.

> At Google, we classify every one of our tests into a size and encourage engineers to always write the smallest possible test for a given piece of functionality.

> The other important constraints on small tests are that they aren’t allowed to sleep, perform I/O operations, or make any other blocking calls.

> … medium tests aren’t allowed to make network calls to any system other than localhost. In other words, the test must be contained within a single machine.

> We mostly reserve large tests for full-system end-to-end tests that are more about validating configuration than pieces of code, and for tests of legacy components for which it is impossible to use test doubles.

> All tests should strive to be hermetic: a test should contain all of the information necessary to set up, execute, and tear down its environment. 

> We have a name for this general philosophy: we call it the Beyoncé Rule. Succinctly, it can be stated as follows: “If you liked it, then you shoulda put a test on it.”

> New episodes are eagerly anticipated and some engineers even volunteer to post the episodes around their own buildings. We intentionally limit each episode to exactly one page, challenging authors to focus on the most important and actionable advice. A good episode contains something an engineer can take back to the desk immediately and try.

> Automated testing is foundational to enabling software to change.

### Unit testing
Add tests when adding new features or behaviors, or fixing a bug. Modify tests when changing behaviors or interfaces…

> When an engineer refactors the internals of a system without modifying its interface, whether for performance, clarity, or any other reason, the system’s tests shouldn’t need to change. 

> At Google, we’ve found that engineers sometimes need to be persuaded that testing via public APIs is better than testing against implementation details. The reluctance is understandable because it’s often much easier to write tests focused on the piece of code you just wrote rather than figuring out how that code affects the system as a whole. Nevertheless, we have found it valuable to encourage such practices, as the extra upfront effort pays for itself many times over in reduced maintenance burden. Testing against public APIs won’t completely prevent brittleness, but it’s the most important thing you can do to ensure that your tests fail only in the event of meaningful changes to your system.

> With state testing, you observe the system itself to see what it looks like after invoking with it. With interaction testing, you instead check that the system took an expected sequence of actions on its collaborators in response to invoking it. Many tests will perform a combination of state and interaction validation. Interaction tests tend to be more brittle than state tests for the same reason that it’s more brittle to test a private method than to test a public method: interaction tests check how a system arrived at its result, whereas usually you should care only what the result is.

> The most common reason for problematic interaction tests is an over reliance on mocking frameworks.

> Two high-level properties that help tests achieve clarity are completeness and conciseness. A test is complete when its body contains all of the information a reader needs in order to understand how it arrives at its result. A test is concise when it contains no other distracting or irrelevant information

> Test Behaviors, Not Methods

> A test’s name should summarize the behavior it is testing. 

> A good trick if you’re stuck is to try starting the test name with the word “should.”

> Instead of being completely DRY, test code should often strive to be DAMP—that is, to promote “Descriptive And Meaningful Phrases.” A little bit of duplication is OK in tests so long as that duplication makes the test simpler and clearer.

DRY:

```java
@Test
public void shouldAllowMultipleUsers() {
  List<User> users = createUsers(false, false);
  Forum forum = createForumAndRegisterUsers(users);
  validateForumAndUsers(forum, users);
}

@Test
public void shouldNotAllowBannedUsers() {
  List<User> users = createUsers(true);
  Forum forum = createForumAndRegisterUsers(users);
  validateForumAndUsers(forum, users);
}

// Lots more tests...

private static List<User> createUsers(boolean... banned) {
  List<User> users = new ArrayList<>();
  for (boolean isBanned : banned) {
    users.add(newUser().setState(isBanned ? State.BANNED : State.NORMAL).build());
  }

  return users;
}

private static Forum createForumAndRegisterUsers(List<User> users) {
  ...
}

...
```

DAMP:

```java
@Test
public void shouldAllowMultipleUsers() {
  User user1 = newUser().setState(State.NORMAL).build();
  User user2 = newUser().setState(State.NORMAL).build();

  Forum forum = new Forum();
  forum.register(user1);
  forum.register(user2);

  assertThat(forum.hasRegisteredUser(user1)).isTrue();
  assertThat(forum.hasRegisteredUser(user2)).isTrue();
}

@Test
public void shouldNotRegisterBannedUsers() {
  User user = newUser().setState(State.BANNED).build();
  Forum forum = new Forum();

  try {
    forum.register(user);
  } catch(BannedUserException ignored) {}

  assertThat(forum.hasRegisteredUser(user)).isFalse();
}
```

### Test doubles
> A seam is a way to make code testable by allowing for the use of test doubles—it makes it possible to use different dependencies for the system under test rather than the dependencies used in a production environment. Dependency injection is a common technique for introducing seams.

> With dynamically typed languages such as Python or JavaScript, it is possible to dynamically replace individual functions or object methods. Dependency injection is less important in these languages because this capability makes it possible to use real implementations of dependencies in tests while only overriding functions or methods of the dependency that are unsuitable for tests.

> Mocking frameworks exist for most major programming languages. At Google, we use Mockito for Java, the googlemock component of Googletest for C++, and unittest.mock for Python.

> At Google, the preference for real implementations developed over time as we saw that overuse of mocking frameworks had a tendency to pollute tests with repetitive code that got out of sync with the real implementation and made refactoring difficult.

> In general, you should perform interaction testing only for functions that are statechanging. Performing interaction testing for non-state-changing functions is usually redundant given that the system under test will use the return value of the function to do other work that you can assert. The interaction itself is not an important detail for correctness, because it has no side effects.

### Large testing
- Functional testing of one or more binaries
- Browser and device testing
- Performance, load, and stress testing
- Deployment configuration testing
- Exploratory testing
- A/B diff (regression) testing
- User acceptance testing (UAT)
- Probers and canary analysis
- Disaster recovery and chaos engineering
- User evaluation

### Version control and branch management
> All of that effort in merging and retesting is pure overhead. The alternative requires a different paradigm: trunk-based development, rely heavily on testing and CI, keep the build green, and disable incomplete/untested features at runtime. Everyone is responsible to sync to trunk and commit; no “merge strategy” meetings, no large/expensive merges.

> … “For every dependency in our repository, there must be only one version of that dependency to choose.”9 For third-party packages, this means that there can be only a single version of that package checked into our repository, in the steady state.10 For internal packages, this means no forking without repackaging/renaming: it must be technologically safe to mix both the original and the fork into the same project with no special effort. This is a powerful feature for our ecosystem: there are very few packages with restrictions like “If you include this package (A), you cannot include other package (B).”

### Code search
> The most frequent use case—about one third of Code Searches—are about seeing examples of how others have done something.

> About 16% of Code Searches try to answer the question of why a certain piece of code was added, or why it behaves in a certain way. Such questions often arise during debugging; for example, why does an error occur under these particular circumstances? An important capability here is being able to search and explore the exact state of the codebase at a particular point in time.

> About 8% of Code Searches try to answer questions around who or when someone introduced a certain piece of code, interacting with the version control system.

### Static analysis
> Through static analysis at Google, we codify best practices, help keep code current to modern API versions, and prevent or reduce technical debt.

### Dependency management
> All else being equal, prefer source control problems over dependency-management problems. If you have the option to redefine “organization” more broadly (your entire company rather than just one team), that’s very often a good trade-off. Source control problems are a lot easier to think about and a lot cheaper to deal with than dependency-management ones.

### Large-scale changes
> At Google, we’ve long ago abandoned the idea of making sweeping changes across our codebase in these types of large atomic changes. Our observation has been that, as a codebase and the number of engineers working in it grows, the largest atomic change possible counterintuitively decreases.

### Continuous integration
> As discussed in Chapter 11, the cost of a bug grows almost exponentially the later it is
caught.
>
> - They must be triaged by an engineer who is likely unfamiliar with the problematic code change.
> - They require more work for the code change author to recollect and investigate the change.
> - They negatively affect others, whether engineers in their work or ultimately the end user.

> We also have a system for flake classification, which uses statistics to classify flakes at a Google-wide level, so engineers don’t need to figure this out for themselves to determine whether their change broke another project’s test (if the test is flaky: probably not).

> So, which tests should be run on presubmit? Our general rule of thumb is: only fast, reliable ones. You can accept some loss of coverage on presubmit, but that means you need to catch any issues that slip by on post-submit, and accept some number of rollbacks. On post-submit, you can accept longer times and some instability, as long as you have proper mechanisms to deal with it.

### Continuous delivery
> You get extraordinary outcomes by realizing that the launch never lands but that it
begins a learning cycle where you then fix the next most important thing, measure how
it went, fix the next thing, etc.—and it is never complete.

> Strive for modular architecture to isolate changes and make troubleshooting easier.

> The investment with the best return, though, is migrating to a microservice architecture, which can empower a large product team with the ability to remain scrappy and innovative while simultaneously reducing risk. In some cases, at Google, the answer has been to rewrite an application from scratch rather than simply migrating it, establishing the desired modularity into the new architecture. Although either of these options can take months and is likely painful in the short term, the value gained in terms of operational cost and cognitive simplicity will pay off over an application’s lifespan of years.

> A key to reliable continuous releases is to make sure engineers “flag guard” all changes. 

> Over the years and across all of our software products, we’ve found that, counterintuitively, faster is safer. The health of your product and the speed of development are not actually in opposition to each other, and products that release more frequently and in small batches have better quality outcomes. They adapt faster to bugs encountered in the wild and to unexpected market shifts. Not only that, faster is cheaper, because having a predictable, frequent release train forces you to drive down the cost of each release and makes the cost of any abandoned release very low.

### Compute as a service
> As your organization grows and your products become more popular, you will grow in all of these axes:
>
> - Number of different applications to be managed
> - Number of copies of an application that needs to run
> - The size of the largest application

# Development Operations (DevOps)

[![Greenkeeper badge](https://badges.greenkeeper.io/MySolace/DevOps.svg)](https://greenkeeper.io/)

[![Build status](https://ci.appveyor.com/api/projects/status/inacxglh4egpjsh9?svg=true)](https://ci.appveyor.com/project/MySolace/devops)

---

### [The Right Tools](./tool-kit.md)

Tools are how we do what we do. There are more and less suited tools for every job. Some like to use everything seperately and scripted together in a commandline. Others might work better in some sort of IDE interface. The toolchains and structures used aren't always the latest and greatest, but rather the practical _best_.

---

### [Fail Intelligently](./successful-failure.md)

Failure is inevitable. Embrace it! Fail fast, fail early, fail often, and absorb every lesson you can. Development is an iterative process, and failure narrows the solution options. The trick is to fail the right way. We employ fault-tolerant systems and redundancy to retain our mission-critical high availability. If something might break, do what you can to break it and then start in on fixing.

---

### [Speed](./faster-pussycat.md)

When a person decides to text in to CTL, they expect to be paired with a [CC]() quickly. Fourteen or fewer minutes often feels like not much waiting when using SMS, but that's a ceiling. We aim to have all texters paired and talking in under five. For that to happen, both the CC team and the software have to be efficient, and the interactions between them fluent and intuitive. A CC can't be spending vital minutes trying to remember _how_ to use the platform.

---

### [Testing](./testing-1-2.md)

The importance of functioning code is paramount, naturally. Nothing happens without that. Functional code isn't always the best it can be right away - it's like editing a story you've written before you publish. Automated tests like [Jenkins]() and [Circle CI]() among others can speed the revision, test, deploy cycles.

---

### [Clouds](./cloudy-with-a-chance-of-meatballs.md)

Scalability and constant deployment and integration are vital to the continued efficiency of our systems. We make extensive use of [AWS]() in particular. By developing and deploying in the cloud, our availability and response speeds are improved greatly.

---

### [Open Standards](./standard-procedure.md)

Over time, developers have shifted toward a culture of keeping software open and easily maintained or altered by other developers later on. The key to accomplishing this is by writing code using accepted consensus standards. These standards do evolve over time, and a part of development is keeping the codebase in step with those standards.

---

### [Code Badges](./badge-and-rank-please.md)

Badges are awesome. At a glance, they can tell somebody inspecting your code what was tested, how that turned out, and a lot more information useful to those seeking small bugs to fix. For the same reason, liberal (but prudent) use of the Github "issues" feature is encouraged. Those with commit priveledges will review and comment or merge any patches submitted for pull requests.

# Development Security Operations (DevSecOps)

### [Security](./dev-sec-ops.md)

Every line of code _must_ be tested and attacked at as high a level of coverage as we can possibly hit. Each bug is something that can be exploited, and any failure of our systems can be a life not saved right now. Features are okay being slow to move in interest of them moving forward safely. Development Security, or DevSecOps, covers much more than just the code itself. Security is a wholistic endeavour and always evolving. Reddit's [/r/devops](https://reddit.com/r/devops/) is a reasonable place to start.

---

### [Access Control](./chaos-control.md)

We're all human. Humans are nice, and so often well meaning. When it comes down to it, we just aren't trustworthy in certain situations. Development Operations is one such environment. Access controls and rights must not rely on policies or enforcement by any means other than proper code and testing.

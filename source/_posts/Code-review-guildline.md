---
title: Code review guildline
date: 2017-02-23 12:39:39
tags:
---
Code reviews, as a company best practice, have been an important part of the software development process. It is must, not optional.

Important benefits of code reviews are learning and sharing.

The developer performing the review learns more about a part of the system that he may not be familiar with. The developer receiving the review gets a valuable critique of their work and learns how to build better software. Both aspects are particularly important for developers that are new to the team.

Additionally, studies have shown that peer reviews are generally more effective than testing at finding software defects. And studies have also shown that code reviews actually increase productivity

Here, I want to share some code review checkpoints:

- Clarity
    - Easy to read
    - Consistent Style
    - Concise
    - Documented
    - Nothing Tricky
    - Appropriate Abstraction
    - No Debug Leftovers
    - No Dumb Comments
- Correctness
    - No Obvious Bugs
    - Unit Tests
    - Test Coverage
    - Thread Safe
- Design
    - Makes Sense
    - Will Integrate
- Reuse
    - No Duplication
    - No Need to Refactor
    - Not redundant
- Configuration and Constants
    - Config actually needed
    - No Magic Numbers
    - No Assumed Defaults
    - Config Documented
- Versioning
    - Backwards Compatible API
    - Backwards Compatible CFG
    - No Deployment Issues
- Dependencies
    - No risky new dependencies
    - No unused dependencies
- Failure Handling
    - Resilient to garbage return values
    - Resilient to garbage input values
    - Edge cases covered
    - Clear exception strategy
    - Null check before for each loop
- Performance
    - No premature optimization
    - No obvious problems with performance
- Instrumentation
    - Appropriate Logging
    - Appropriate Log levels
    - Appropriate Metrics

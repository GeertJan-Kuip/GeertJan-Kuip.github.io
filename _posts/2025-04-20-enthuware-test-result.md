## Enthuware test result

To prepare for 1Z0-819 I bought access to the online mock tests of [Enthuware](https://enthuware.com/). I did a test from [Udemy](https://www.udemy.com/) as well. From both I like the convenience of doing it on online forms, which is better than the ebook pen and paper tests of the book I learn from.

For the Udemy test I scored 66% some days ago, for the first Enthuware test done this morning I scored 64%. Both are better than the scores I get for the tests from the book and I'm still below 68%. Work to do.

### Review

This is the review of my wrong answers on the first Enthuware test. I marked the right answers first and my (wrong) answers second:

**Question 5** - _Which of the following are valid command line options and their one character shortcuts related to the Java module system? Select 2._

- [ ] - [ ]\--module -p
- [x] - [x]\--module-path -p
- [ ] - [ ]\--module-source-path -s
- [ ] - [ ]---module-source-path -m
- [ ] - [x]\--list-modules -l
- [ ] - [ ]\--show-module-resolution -s
- [x] - [ ]\--module -m

I simply had little clue, just started the Modules chapter.

**Question 8** -_Which of the following annotations are retained for run time? Select 3._

- [ ] - [x]@SuppressWarnings
- [ ] - [ ]@Override
- [x] - [x]@SafeVarargs
- [x] - [ ]@FunctionalInterface
- [x] - [x]@Deprecated

@SuppressWarnings and @Override are defined with @Retention(SOURCE), the others with @Retention(RUNTIME). I thought @SuppressWarnings would be a better candidate, at least it would have the same retention policy as @SafeVarargs given their similar role of suppressors. There seems to be some randomness in it, why would @FunctionalInterface need to be available at runtime?

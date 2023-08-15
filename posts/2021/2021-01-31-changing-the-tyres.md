<!--
.. title: Changing the tyres while the car is moving
.. slug: changing-the-tyres
.. date: 2021-01-31 00:00:00
.. tags: python,testing
.. category: 
.. link: 
.. description: 
.. type: text
-->

I am currently working on a project where I need to migrate a legacy test suite from nose to pytest. The codebase has about 7,000 lines of test code. That isn't an enormous test suite, but I'm the only person who will be working on it. It will take me a while to get through them all because moving from one test runner to another will require changes to fixtures, factories, etc and maybe some light codebase refactoring as well as just reviewing/updating the test code itself. I also need to balance this with continuing to deliver bugfix and feature work. I can't block all delivery on the test suite migration, so here's a pattern:

- Split the test suite into two dirs: `/tests/nose` and `/tests/pt`
- Migrate one module of test code at a time
- Have the CI build run the old tests with nose and the new ones with pytest and merge the coverage reports:

```yaml
- name: Run nose tests
  run: |
    nosetests
      --with-coverage \
      --cover-package=dir \
      --cover-erase \
      path/to/tests/nose

- name: Run pytest tests
  run: |
    pytest \
      --cov=dir \
      --cov-append \
      path/to/tests/pt
```

- When I rewrite the last test module left in `/tests/nose`, we can finally drop nose, delete any nose-specific test helpers and move all the tests from `/tests/pt` back to `/tests`.

This approach buys me several useful things:

- I can tackle the test suite migration one module at a time
- I can submit small pull requests that are easy to review
- I can start writing tests for any new features or bugfixes in pytest right now
- I can minimise the chance of conflicts between test suite migration and bugfix/feature delivery

This is a very simple example of a high-level general pattern for managing non-trivial migrations (like moving a project from one web framework to another). This can be applied to larger more complex migrations and enables us to "change the tyres while the car is moving" (a phrase I stole from my colleague [Adrià](https://amercader.net/)):

- Set up a structure that allows both `$OLD_BORING_THING` and `$NEW_SHINY_THING` to run at the same time
- Gradually move code from `$OLD_BORING_THING` to `$NEW_SHINY_THING`
- When we migrate the last bit of code from `$OLD_BORING_THING` to `$NEW_SHINY_THING`, remove the structure that allows `$OLD_BORING_THING` and `$NEW_SHINY_THING` to co-exist and delete `$OLD_BORING_THING`
- Continue to deliver our roadmap in parallel to the migration

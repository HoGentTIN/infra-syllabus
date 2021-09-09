# Continuous Integration/Delivery


- Developers will usually push to a Git server that is maintained internally. When a new commit is pushed, it will trigger Jenkins to start the build process.
- First, a linter and/or static code analysis may be run to ensure that all code complies with style guidelines.
- After that, the code can be built.
- Next, unit tests can be executed.
- If that succeeds, the application can be packaged and launched in a test environemt.
- In the test environment, additional integration/acceptance tests can be run.
- If all that succeeds, the packaged application can be deployed in the production environment.

cfr. Reflectie in labo-opgave

Iets over *trunk based development*: het concept van branches staat haaks op de filosofie van CI/CD.
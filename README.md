# central-publishing-maven-plugin-bug

The minimal reproducer for `central-publishing-maven-plugin` bug. It's
not really a bug but a result of the plugin's logic. 

When using the plugin in a multi-module project, the `central-publishing-maven-plugin`
will fail to publish the artifacts if the last module sets `skipPublishing` to `true`.

The setup is as follows:
1. Parent module declares usage of `central-publishing-maven-plugin` and sets `skipPublishing` to a value of a variable (e.g. `maven.deploy.skip`).
2. This variable is resolved for each submodule independently.
3. First submodule (called `deployed`) sets the variable to false and should be deployed.
4. Second submodule (called `not-deployed`) sets the variable to true and should be deployed.

This should be a common case, e.g., the first module is a public api, and the second module is an api implementation distributed as a docker image.

The bug is that the `central-publishing-maven-plugin` does not do the final publishing until processing the last module in the project (`PublishMojo::doExecute` checks `isThisLastProjectWithThisMojoInExecution`) and then evaluates the `skipPublishing` value. If that last module is skipped, the `central-publishing-maven-plugin` will not publish anything.

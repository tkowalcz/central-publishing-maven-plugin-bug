# central-publishing-maven-plugin-bug

The minimal reproducer for `central-publishing-maven-plugin` bug. It's
not really a bug but a result of the plugin's logic. 

When using the plugin in a multi-module project, the `central-publishing-maven-plugin`
will fail to publish the artifacts if the last submodule sets `skipPublishing` to `true`.

The setup is as follows:
1. Parent module declares usage of `central-publishing-maven-plugin` and sets `skipPublishing` to a value of a variable (e.g. `maven.deploy.skip`).
    ```xml
    <plugin>
        <groupId>org.sonatype.central</groupId>
        <artifactId>central-publishing-maven-plugin</artifactId>
        <version>0.8.0</version>
        <configuration>
            <skipPublishing>${maven.deploy.skip}</skipPublishing>
        </configuration>
    </plugin>
   ```
   
2. This variable is resolved for each submodule independently.
3. First submodule (called `deployed`) sets the variable to false and should be deployed.
4. Second submodule (called `not-deployed`) sets the variable to true and should be deployed.
5. The second submodule depends on the first one to make sure it is built last.

This should be a common case, e.g., the first submodule is a public api, and the second submodule is an api implementation distributed as a docker image.

The bug is that the `central-publishing-maven-plugin` does not do the final publishing until processing the last submodule in the project (`PublishMojo::doExecute` checks `isThisLastProjectWithThisMojoInExecution`) and then evaluates the `skipPublishing` value. If that last submodule is skipped, the `central-publishing-maven-plugin` will not publish anything.

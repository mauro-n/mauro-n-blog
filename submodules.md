a Git submodule is essentially a pointer. Your parent repository doesn't track the submodule's files; it just records a specific commit hash from that external repository.

Here is exactly what happens in the scenarios you mentioned, and why the community has such strong opinions on the topic.

1. What if I modify the project I cloned (the submodule)?
Because the submodule is a fully independent Git repository nested inside your parent repo folder, modifying it requires a specific workflow. It is not as simple as just saving and committing in the parent.

The Detached HEAD trap: By default, a submodule checks out a specific commit, not a branch. If you modify code and commit immediately, you will be in a "detached HEAD" state.

The correct workflow:

cd into the submodule directory.

Run git checkout main (or whichever branch you are working on).

Make your code changes.

Commit and push those changes to the submodule’s remote.

cd back up to your parent repository.

Commit the new submodule pointer (the new commit hash) and push the parent repository.

If you skip step 4 and only commit the pointer in the parent repo, you will break the build for everyone else (more on that below).

2. What happens when another person clones my repo?
This is where the friction usually starts for teams.

When a colleague (or your CI/CD pipeline) runs git clone <your-repo>, Git pulls down the parent repository but leaves the submodule folder completely empty.

To actually get the code, they must run:
git submodule update --init --recursive

Alternatively, they could have cloned the parent initially using git clone --recurse-submodules <url>.

The dreaded "Forgot to Push" error: If you updated the submodule locally, committed the new pointer to the parent, but forgot to push the actual submodule code to its remote (skipping step 4 from above), your teammate will clone the parent, try to update the submodule, and Git will throw a fatal error because it cannot find the commit hash you pointed to.

3. What is the relation to monorepos (Turborepo, etc.)?
Submodules were an early attempt at solving the "multi-repo vs. monorepo" problem. They try to give you the independent history of microservices, but with the local file-system convenience of a monorepo.

Modern monorepo tools like Turborepo, Nx, or Lerna approach the problem entirely differently:

The Monorepo Way: All your code (frontend, backend, shared packages) lives in one single Git repository. There are no pointers. It is just one big tree of files.

The Tooling: Because a massive codebase can be slow to build, tools like Turborepo step in. They understand the dependency graph of your workspaces. If you change a UI component, Turborepo knows to only rebuild the apps that depend on that component, pulling the rest from a cache.

If you are trying to share internal packages within a single product ecosystem, modern monorepo tooling is vastly superior to stitching repositories together with submodules. It keeps the Git workflow incredibly simple (just git commit and git push).

4. Why do people talk bad about submodules and suggest subtrees?
People dislike submodules because the developer experience (DX) is fragile. Forgetting to push the submodule, dealing with detached HEADs, and forcing teammates to remember specific update commands creates constant workflow hiccups.

Git Subtrees are often recommended as an alternative because they solve the "pointer" problem.

Instead of storing a link to a commit, git subtree physically merges the history and files of the external repository into your parent repository.

To anyone cloning your repository, the subtree just looks like regular files. They don't need to run any special --init commands.

You can still push updates from that folder back to the upstream repository if you need to, but the day-to-day work doesn't require juggling two separate Git states at once.

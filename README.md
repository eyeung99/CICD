protoc -I=. --gofast_out=plugins=grpc:. ./example.proto

## Setting up GitHub Actions for Your Project

GitHub Actions for our Go project. It is in Go, and it can be in any languages like Node.js, Python, Java etc. Try it out on your own repository.

**What are GitHub Actions?**
* GitHub Actions help you automate your software development workflows in the same place you store code and collaborate on pull requests and issues
* You can write individual tasks, called actions, and combine them to create a custom workflow. Workflows are custom automated processes that you can set up in your repository to build, test, package, release, or deploy any code project on GitHub

* To set up a complete pipeline, you can find a variety of actions contributed by the GitHub community in GitHub Marketplace

**Setting up the Workflow**

A workflow is a custom automated process that you can include in your repository to build, test, and deploy your source codes. You can have more than one workflow and it must be stored in `.github/workflows` folder in the root directory of the source code.

Build your workflow file which can be either .yaml or .yml.

1. Name the workflow (the name of your choice)

```
name: CICD
```

2. Configure the events to trigger the workflow. It can be either through events on GitHub (for example a pull request or push) or based on a schedule. In the example below, we will trigger the workflow on push and pull requests on branches with the names ‘main’, ‘staging’, ‘develop’.

```
on:
 push:
   branches:
   - main
   - staging
   - develop
 pull_request:
   branches:
   - main
   - staging
   - develop
```
3. Creating the first job. In this example, we will create two jobs, one is called ‘build’ and the next one will be called ‘deploy’. Jobs can be run sequentially or in parallel. And each job must have steps.

```
jobs:
 # The “build” workflow
 build:
```

4. Choose a runner. Workflow runs on a runner, either GitHub-hosted runners or self-hosted runners. With GitHub-hosted runners, we can also specify the version and type of virtual host machine.

```
runs-on: ubuntu-latest
```

5. Start the job with checkout action. This is a standard action to be added at the beginning of the job to get a copy of the repository’s source code.

```
steps:
 - uses: actions/checkout@v2
```

6. Setup the environment to use Go. Also, we have to specify the version of Go that we will be using.

```
- name: Setup Go
 uses: actions/setup-go@v2
 with:
   go-version: ‘1.15.x’
```

7. Install all the dependencies.

```
- name: Install dependencies
 run: |
   go version
   go get -u golang.org/x/lint/golint
```

8. Package and build the application.

```
- name: Run build
 run: go build .
```

9. Run vet and lint (Vet examines Go source code and reports suspicious constructs, such as Printf calls whose arguments do not align with the format string. Vet uses heuristics that do not guarantee all reports are genuine problems, but it can find errors not caught by the compilers. Vet is normally invoked through the go command; golint prints out style mistakes)

```
- name: Run vet & lint
 run: |
   go vet .
   golint .
```

10. Run gofmt to format the code with gofmt. Check for modified files, then push changes. Enter your name and email address for notification

```
   - name: Format
      run: gofmt -s -e -w .
    - name: Check for modified files
      id: git-check
      run: echo ::set-output name=modified::$(if git diff-index --quiet HEAD --; then echo "false"; else echo "true"; fi)
    - name: Push changes
      if: steps.git-check.outputs.modified == 'true'
      run: |
        git config --global user.name 'Estella Yeung'
        git config --global user.email 'estellayeung@hotmail.com'
        git remote set-url origin https://x-access-token:${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}
        git branch temp-branch
        git checkout main
        git commit -am "Automated changes"
        git merge temp-branch        
        git push origin main
```


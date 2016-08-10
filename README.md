# Jenkinsfile Recipe
This repository lists useful code snippets for Jenkinsfile. Even with step references, snippet generator, and stack overflow, it is hard to learn what kind of tasks are available in Jenkinsfile. IE, figure out how to mark the github commit status using Jenkinsfile was very painful for me. So I hope this recipe can be helpful to those who just began learning about Jenkinsfile.

If you find a useful snippet is missing, please make a PR!


### Set up related recipes
#### To set a commit status in github to be `in progress`
```
step([$class: 'GitHubSetCommitStatusBuilder'])
```
- Tested in Jenkins ^2.1

#### To run the build with the merged code
```
checkout(
    scm: [
        $class: 'GitSCM',
        branches: [
            [name: "refs/heads/${env.BRANCH_NAME}"]
        ],
        extensions: [
            [$class: 'PreBuildMerge', options: [fastForwardMode: 'FF', mergeRemote: 'origin', mergeTarget: branchToMerge]]
        ],
        userRemoteConfigs: [
            [credentialsId: credentialsId, url: gitUrl]
        ]
    ]
)
```
- Tested in Jenkins ^2.14
- Unclear whether same could be used for subversion

### Build related recipes
#### To use a shell command stdout in Jenkinsfile
```
sh(script: script, returnStdout: true)
```
- Tested in Jenkins ^2.16

#### To catch errors per stage
```
def handleStageFailure(String stageName, Closure stageDefinition) {
    stage stageName
    try {
        stageDefinition()
    } catch (err) {
        setCurrentBuildFailureStatus("${stageName} failed: ${err}")
        throw err
    }
}
```
- Tested in Jenkins ^2.14

#### To not include credentials in SCM
```
withCredentials([
    [$class: 'StringBinding', credentialsId: credentialsId, variable: credentialsEnvId],
]) {
    //Stuff to do
}
```
- Tested in Jenkins ^2.14

### Tear down related recipes
#### To make build statuses descriptive
```
currentBuild.description = buildStatus
```
- Tested in Jenkins ^2.1

#### To archive Junit xml files
```
step([$class: 'JUnitResultArchiver', testResults: testResultsPath])
```
- `testResultsPath` is in the form of `output/**/*.xml`
- Tested in Jenkins ^2.16
- Needs https://wiki.jenkins-ci.org/display/JENKINS/JUnit+Plugin
- Throws an exception when there is no test results in the path

#### To archive other build artifacts
```
archive excludes: 'excludesPath', includes: 'includesPath'
```
- `excludesPath` and `includesPath` are in the form of `output/**.xml`.
- Tested in Jenkins ^2.14

#### To clean up the directory before/after the build
```
deleteDir()
```
- Tested in Jenkins ^2.1

#### To set a commit status in github based on `currentBuild.status` after the build is finished
```
step([
    $class: 'GitHubCommitStatusSetter',
    errorHandlers: [[$class: 'ShallowAnyErrorHandler']],
    statusResultSource: [
        $class: 'ConditionalStatusResultSource',
        results: [
            [$class: 'BetterThanOrEqualBuildResult', result: 'SUCCESS', state: 'SUCCESS', message: currentBuild.description],
            [$class: 'BetterThanOrEqualBuildResult', result: 'FAILURE', state: 'FAILURE', message: currentBuild.description],
            [$class: 'AnyBuildResult', state: 'FAILURE', message: 'Loophole']
        ]
    ]
])
```
- Tested in Jenkins ^2.1

# License
MIT

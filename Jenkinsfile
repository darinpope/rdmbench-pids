// Jenkins CI for the RDM manufacturer PID tables.
//
// One agent, one job: every table loads, names a source, stays inside the
// manufacturer-specific PID range, and sits where a bench would look for it.
//
// The checker is RDMBench's own `rdmbench-cli` — the same loader that reads
// these files on a bench — rather than a validator written a second time for
// this repo. That is only possible because this is our Jenkins rather than a
// public CI sandbox: the agent can have the tool, and the data stays checkable
// by the code that consumes it. Nothing here needs the RDMBench *sources*,
// only the built binary on PATH.
//
// The commands are written out here rather than run through `task`, unlike
// RDMBench. `task` reads Taskfile.yml from the checkout, and on a pull request
// that file is the pull request's own: editing it would run any command on the
// agent that builds RDMBench. A Jenkinsfile is the one file Jenkins can take
// from the base branch instead, so what CI runs has to live in it. Taskfile.yml
// keeps the same commands for a maintainer at their own desk — change both
// together. Only `rdmbench-cli` and git need to be on PATH.
//
// The manifest and its signature are checked on main — CI only verifies; the
// signing key is the maintainer's and never on an agent, or a merge would be a
// release. They are NOT checked on a pull request: contributors have
// no way to regenerate it (the tool is not public) and are asked not to touch
// it, so a maintainer regenerates it when merging and this job is what catches
// a maintainer who forgot.
//
// ADJUST: the agent label below to match your fleet.

pipeline {
    agent { label 'darin-m2-studio' }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '30'))
        timeout(time: 10, unit: 'MINUTES')
    }

    stages {
        stage('versions') {
            steps {
                sh 'rdmbench-cli --version'
                sh 'git --version'
            }
        }

        stage('validate') {
            steps {
                sh 'rdmbench-cli validate-tables tables'
                sh 'rdmbench-cli validate-profiles profiles'
            }
        }

        stage('manifest') {
            when { not { changeRequest() } }
            steps { sh 'rdmbench-cli pid-manifest tables --check' }
        }
    }
}

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
// Command bodies live in Taskfile.yml, not here — same arrangement as
// RDMBench, so the checks a maintainer runs by hand are the checks that gate a
// merge. Task must be on PATH (https://taskfile.dev).
//
// The manifest is checked on main and NOT on a pull request: contributors have
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
            steps { sh 'task versions' }
        }

        stage('validate') {
            steps { sh 'task validate' }
        }

        stage('manifest') {
            when { not { changeRequest() } }
            steps { sh 'task manifest:check' }
        }
    }
}

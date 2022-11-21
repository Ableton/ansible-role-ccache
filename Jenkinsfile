library(identifier: 'ableton-utils@0.22', changelog: false)
library(identifier: 'groovylint@0.13', changelog: false)
library(identifier: 'python-utils@0.13', changelog: false)


devToolsProject.run(
  setup: { data ->
    // Temporary workaround for molecule's awful handling of roles and collections. It
    // insists on installing everything to ~/.cache/ansible-compat, which causes problems
    // on our CI system once it discovers that roles with different versions are there.
    sh 'rm -rf $HOME/.cache/ansible-compat'

    Object venv = pyenv.createVirtualEnv(readFile('.python-version'))
    venv.inside {
      sh 'pip install -r requirements-dev.txt'
      data['rolesPath'] = "${env.WORKSPACE}/.ansible/roles"
      sh "ansible-galaxy install --force --roles-path ${data.rolesPath}" +
        ' --role-file requirements.yml'
      sh "ansible-galaxy install --no-deps --roles-path ${data.rolesPath}" +
        " git+https://github.com/${params.JENKINS_REPO_SLUG},${params.JENKINS_COMMIT}"
    }
    data['venv'] = venv
  },
  test: { data ->
    data.venv.inside {
      parallel(failFast: false,
        'ansible-lint': {
          sh(
            label: 'ansible-lint',
            script: 'ansible-lint --strict --offline -c .ansible-lint.yml',
          )
        },
        groovylint: { groovylint.checkSingleFile(path: './Jenkinsfile') },
        molecule: {
          withEnv(["ANSIBLE_ROLES_PATH=${data.rolesPath}"]) {
            sh 'molecule --debug test --all'
          }
        },
        yamllint: { sh 'yamllint --strict .' },
      )
    }
  },
  deployWhen: { devToolsProject.shouldDeploy(defaultBranch: 'main') },
  deploy: { data ->
    String versionNumber = readFile('VERSION').trim()
    version.tag(versionNumber)

    List repoSlugParts = params.JENKINS_REPO_SLUG.split('/')
    String repoOrg = repoSlugParts[0]
    String repoName = repoSlugParts[1]
    withCredentials([
      string(credentialsId: 'ansible-galaxy-api-key', variable: 'API_KEY')
    ]) {
      data.venv.run(
        label: 'Publish to Ansible Galaxy',
        script: "ansible-galaxy role import --branch ${versionNumber}" +
          ' --api-key $API_KEY' +  // avoid exposing the credential in the build log
          " ${repoOrg} ${repoName}",
      )
    }
  },
)

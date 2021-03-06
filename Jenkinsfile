
node {
  git 'https://github.com/ipcrm/rgbank'

  stage 'Lint and unit tests'
  withEnv(['PATH=/usr/local/bin:$PATH']) {
    sh """
      source ~/.bash_profile
      rbenv global 2.3.1
      eval "\$(rbenv init -)"
      bundle install
      bundle exec rspec spec/
    """
  }

  stage 'Build and package'
  def version = env.BUILD_ID
  sh 'tar -czf rgbank-build-$BUILD_ID.tar.gz -C src .'
  archive "rgbank-build-${version}.tar.gz"
  archive "rgbank.sql"
  step([$class: 'CopyArtifact', filter: "rgbank-build-${version}.tar.gz", fingerprintArtifacts: true, projectName: env.JOB_NAME, selector: [$class: 'SpecificBuildSelector', buildNumber: env.BUILD_ID], target: '/var/www/html/builds/rgbank'])
  step([$class: 'CopyArtifact', filter: "rgbank.sql", fingerprintArtifacts: true, projectName: env.JOB_NAME, selector: [$class: 'SpecificBuildSelector', buildNumber: env.BUILD_ID], target: '/var/www/html/builds/rgbank'])

  def hostaddress = 'jenkins.demo.lan'

  stage 'Deployment Test'
  puppet.hiera scope: 'beaker', key: 'rgbank-build-version', value: version
  puppet.hiera scope: 'beaker', key: 'rgbank-build-path', value: "http://" + hostaddress + "/builds/rgbank/rgbank-build-${version}.tar.gz"
  puppet.hiera scope: 'beaker', key: 'rgbank-mock-sql-path', value: "http://" + hostaddress + "/builds/rgbank/rgbank.sql"
  build job: 'puppetlabs-rgbank-spec', parameters: [[$class: 'StringParameterValue', name: 'COMMIT', value: env.rgbank_module_ver]]

  puppet.credentials 'pe-access-token'

  stage 'Deploy to dev'
  puppet.hiera scope: 'dev', key: 'rgbank-build-version', value: version
  puppet.hiera scope: 'dev', key: 'rgbank-build-path', value: "http://" + hostaddress + "/builds/rgbank/rgbank-build-${version}.tar.gz"
  puppet.hiera scope: 'dev', key: 'rgbank-mock-sql-path', value: "http://" + hostaddress + "/builds/rgbank/rgbank.sql"
  puppet.job 'dev', application: 'Rgbank'

  stage 'Dev Acceptance Test'
  withEnv(['PATH=/usr/local/bin:$PATH']) {
    sh """
      curl -o /dev/null --silent --head --write-out '%{http_code}\n' http://rgbank-dev.demo.lan/|grep 200 &> /dev/null
      curl -s http://rgbank-dev.demo.lan/|grep 'RG Bank' &>/dev/null
    """
  }

  stage 'Deploy to staging'
  puppet.hiera scope: 'staging', key: 'rgbank-build-version', value: version
  puppet.hiera scope: 'staging', key: 'rgbank-build-path', value: "http://" + hostaddress + "/builds/rgbank/rgbank-build-${version}.tar.gz"
  puppet.hiera scope: 'staging', key: 'rgbank-mock-sql-path', value: "http://" + hostaddress + "/builds/rgbank/rgbank.sql"
  puppet.job 'staging', application: 'Rgbank'

  stage 'Staging Acceptance Test'
  withEnv(['PATH=/usr/local/bin:$PATH']) {
    sh """
      curl -o /dev/null --silent --head --write-out '%{http_code}\n' http://rgbank-dev.demo.lan/|grep 200 &> /dev/null
      curl -s http://rgbank-dev.demo.lan/|grep 'RG Bank' &>/dev/null
    """
  }

  stage 'Noop production run'
  puppet.hiera scope: 'production', key: 'rgbank-build-version', value: version
  puppet.hiera scope: 'production', key: 'rgbank-build-path', value: "http://" + hostaddress + "/builds/rgbank/rgbank-build-${version}.tar.gz"
  puppet.hiera scope: 'production', key: 'rgbank-mock-sql-path', value: "http://" + hostaddress + "/builds/rgbank/rgbank.sql"
  puppet.job 'production', noop: true, application: 'Rgbank'

  stage 'Deploy to production'
  input "Ready to deploy to production?"
  puppet.job 'production', application: 'Rgbank'

}

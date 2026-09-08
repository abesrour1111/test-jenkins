pipeline {
agent any
options {
skipDefaultCheckout(true)
timestamps()
disableConcurrentBuilds()
buildDiscarder(logRotator(numToKeepStr: '10'))
}
environment {
APP_NAME = 'demo-powershell'
BUILD_DIR = 'build-output'
PACKAGE_DIR = 'package'
}
stages {
stage('Checkout') {
steps {
checkout scm
}
}
stage('Préparation') {
steps {
powershell '''
$ErrorActionPreference = 'Stop'
Write-Host "Préparation du build $env:BUILD_NUMBER"
New-Item -Path $env:BUILD_DIR -ItemType Directory -Force | Out-Null
New-Item -Path $env:PACKAGE_DIR -ItemType Directory -Force | Out-Null
$source = Join-Path $env:WORKSPACE 'application.txt'
if (-not (Test-Path -Path $source -PathType Leaf)) {
throw "Le fichier source est absent : $source"
}
Copy-Item -Path $source -Destination $env:BUILD_DIR -Force
Write-Host "Fichier copié vers $env:BUILD_DIR"
'''
}
}
stage('Validation') {
steps {
powershell '''
$ErrorActionPreference = 'Stop'
$source = Join-Path $env:BUILD_DIR 'application.txt'
$content = Get-Content -Path $source -Raw
if ([string]::IsNullOrWhiteSpace($content)) {
throw 'Le fichier application.txt est vide.'
}
if ($content -notmatch 'Application de démonstration Jenkins') {
throw 'Le contenu attendu n’a pas été trouvé.'
}
Write-Host 'Validation du contenu réussie.'
'''
}
}
stage('Test') {
steps {
   powershell '''
$ErrorActionPreference = 'Stop'
$testDir = Join-Path $env:BUILD_DIR 'tests'
New-Item -Path $testDir -ItemType Directory -Force | Out-Null
$testXml = @"
<?xml version="1.0" encoding="UTF-8"?>
<testsuite name="PowerShellTests" tests="1" failures="0" errors="0" time="0.1">
<testcase classname="DemoApplication" name="ApplicationFileExists" time="0.1" />
</testsuite>
"@
$testPath = Join-Path $testDir 'TEST-demo.xml'
$testXml | Set-Content -Path $testPath -Encoding UTF8
Write-Host 'Test automatisé réussi.'
'''
}
}
stage('Packaging') {
steps {
powershell '''
$ErrorActionPreference = 'Stop'
$packageFile = Join-Path $env:PACKAGE_DIR "$env:APP_NAME-$env:BUILD_NUMBER.zip"
if (Test-Path -Path $packageFile) {
Remove-Item -Path $packageFile -Force
}
Compress-Archive -Path (Join-Path $env:BUILD_DIR '*') `
-DestinationPath $packageFile -Force
Write-Host "Package créé : $packageFile"
'''
}
}
}
post {
always {
    archiveArtifacts artifacts: 'package/*.zip, build-output/**/*.xml',
allowEmptyArchive: false,
fingerprint: true
junit testResults: 'build-output/**/*.xml',
allowEmptyResults: false
}
success {
powershell '''
Write-Host "Pipeline réussi : $env:JOB_NAME #$env:BUILD_NUMBER"
'''
}
failure {
powershell '''
Write-Host "Pipeline en échec : consulter la Console Output"
'''
}
}
}

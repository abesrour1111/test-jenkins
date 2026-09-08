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

                    Write-Host 'Préparation du projet'
                    New-Item -ItemType Directory -Path 'build-output' -Force | Out-Null
                    New-Item -ItemType Directory -Path 'package' -Force | Out-Null

                    if (-not (Test-Path 'application.txt')) {
                        Write-Error 'Le fichier application.txt est introuvable.'
                    }

                    Copy-Item 'application.txt' 'build-output/application.txt' -Force
                    Write-Host 'Fichier copié dans build-output.'
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

                    if ($content -notmatch 'Application de Jenkins') {
                        throw 'Le contenu attendu est absent.'
                    }

                    Write-Host 'Validation du contenu réussie.'
                '''
            }
        }

        stage('Test') {
            steps {
                powershell '''
                    $ErrorActionPreference = 'Stop'

                    $source = Join-Path $env:BUILD_DIR 'application.txt'
                    if (-not (Test-Path -Path $source -PathType Leaf)) {
                        throw 'Le test a échoué : application.txt est absent.'
                    }

                    $lineCount = @(Get-Content -Path $source).Count
                    if ($lineCount -lt 2) {
                        throw 'Le test a échoué : le fichier doit contenir au moins deux lignes.'
                    }

                    Write-Host "Test réussi : $lineCount lignes ont été vérifiées."
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

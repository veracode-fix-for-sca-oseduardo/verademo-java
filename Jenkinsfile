//
// This is an example of using VeraDemo Java test application with the Veracode Static scanner.  
//

pipeline {
    agent any

    environment {
        VERACODE_APP_NAME = 'Verademo'      // App Name in the Veracode Platform
    }

    stages{
        stage ('environment verify') {
            steps {
                script {
                    if (isUnix() == true) {
                        sh 'pwd'
                        sh 'ls -la'
                        sh 'echo $PATH'
                    }
                    else {
                        bat 'dir'
                        bat 'echo %PATH%'
                    }
                }
            }
        }

        stage ('build') {
            steps {
                withMaven(maven:'maven-3') {
                    script {
                        if(isUnix() == true) {
                            sh 'mvn -f app clean package'
                        }
                        else {
                            bat 'mvn -f app clean package'
                        }
                    }
                }
            }
        }

        stage ('Veracode scan') {
            steps {
                script {
                    if(isUnix() == true) {
                        env.HOST_OS = 'Unix'
                    }
                    else {
                        env.HOST_OS = 'Windows'
                    }
                }

                echo 'Veracode scanning'
                withCredentials([ usernamePassword ( 
                    credentialsId: 'veracode_login', usernameVariable: 'VERACODE_API_ID', passwordVariable: 'VERACODE_API_KEY') ]) {
                        // fire-and-forget 
                        veracode applicationName: "${VERACODE_APP_NAME}", criticality: 'VeryHigh', debug: true, fileNamePattern: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: 'Jenkins-${BUILD_NUMBER}', uploadExcludesPattern: '', uploadIncludesPattern: 'app/target/verademo.war', vid: "${VERACODE_API_ID}", vkey: "${VERACODE_API_KEY}"

                        // wait for scan to complete (timeout: x)
                        //veracode applicationName: '${VERACODE_APP_NAME}'', criticality: 'VeryHigh', debug: true, timeout: 20, fileNamePattern: '', pHost: '', pPassword: '', pUser: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: "${BUILD_TAG}", uploadExcludesPattern: '', uploadIncludesPattern: 'target/verademo.war', vid: '${VERACODE_API_ID}', vkey: '${VERACODE_API_KEY}'
                    }      
            }
        }

// the above steps are the bare minimum.
// below are some additional steps that are commonplace

        stage ('Veracode SCA') {
            steps {
                echo 'Veracode SCA'
                withCredentials([ string(credentialsId: 'SCA_Token', variable: 'SRCCLR_API_TOKEN')]) {
                    withMaven(maven:'maven-3') {
                        script {
                            if(isUnix() == true) {
                                sh "curl -sSL https://download.sourceclear.com/ci.sh | sh -s -- scan app"

                                // debug, no upload
                                //sh "curl -sSL https://download.sourceclear.com/ci.sh | DEBUG=1 sh -s -- scan --no-upload"
                            }
                            else {
                                powershell '''
                                            Set-ExecutionPolicy AllSigned -Scope Process -Force
                                            $ProgressPreference = "silentlyContinue"
                                            iex ((New-Object System.Net.WebClient).DownloadString('https://download.srcclr.com/ci.ps1'))
                                            srcclr scan app
                                            '''
                            }
                        }
                    }
                }
            }
        }

        // Currently only works on *nix
        stage ('Veracode container scan') {
            steps {
                echo 'Veracode container scanning'
                withCredentials([ usernamePassword ( 
                    credentialsId: 'veracode_login', usernameVariable: 'VERACODE_API_KEY_ID', passwordVariable: 'VERACODE_API_KEY_SECRET') ]) {
                        script {
                            if(isUnix() == true) {
                                sh '''
                                    curl -fsS https://tools.veracode.com/veracode-cli/install | sh
                                    ./veracode scan --type directory --source . --format table
                                    '''
                            }
                            else {
                                powershell '''
                                            Set-ExecutionPolicy AllSigned -Scope Process -Force
                                            $ProgressPreference = "silentlyContinue"
                                            iex ((New-Object System.Net.WebClient).DownloadString('https://tools.veracode.com/veracode-cli/install.ps1'))
                                            $VERACODE_CLI = Get-Command veracode | Select-Object -ExpandProperty Definition
                                            Write-Host "##vso[task.setvariable variable=VERACODE_CLI]$VERACODE_CLI"
                                            # Check if MSBuild is already available in PATH
                                            try {
                                            $msbuildAlreadyExists = Get-Command msbuild.exe -ErrorAction SilentlyContinue
                                            } catch {
                                            $msbuildAlreadyExists = $false
                                            }
                                            Write-Host "MSBuild install check: $msbuildAlreadyExists"
                                            if (-not $msbuildAlreadyExists) {
                                            $vswherePath = "${env:ProgramFiles(x86)}\\Microsoft Visual Studio\\Installer\\vswhere.exe"
                                                # Use a fallback check instead of Test-Path for broader compatibility
                                            try {
                                                if ([System.IO.File]::Exists($vswherePath)) {
                                                Write-Host "vswherePath install check: $vswherePath"
                                                $msbuildPath = & $vswherePath -latest -requires Microsoft.Component.MSBuild -find MSBuild\\**\\Bin\\MSBuild.exe
                                                if ($msbuildPath) {
                                                    $msbuildDir = [System.IO.Path]::GetDirectoryName($msbuildPath)
                                                    Write-Host "msbuildDir install check: $msbuildDir"
                                                    #$env:PATH = $msbuildDir + ";" + $env:PATH
                                                    if ($msbuildDir -and -not ($env:PATH.Split(";") | Where-Object { $_.TrimEnd('\\') -ieq $msbuildDir.TrimEnd('\\') })) {
                                                    $env:PATH = $msbuildDir + ";" + $env:PATH
                                                    Write-Host "msbuild path added: $env:PATH"
                                                    }
                                                }
                                                }
                                            } catch {
                                                Write-Host "ProgressPreference catch block: $ProgressPreference"
                                                # Silently ignore errors
                                            }
                                            }
                                            & $VERACODE_CLI scan --type directory --source . --format table
                                            '''
                            }
                        }
                    }
            }
        }
    }
}

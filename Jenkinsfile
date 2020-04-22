def myRoot = "SomeRoot";
def modules = [
    [
        name: 'module1',
        root: "${myRoot}",
        visibility: "private",
        port: 80
    ], 
    [
        name: 'module2',
        root: "${myRoot}",
        visibility: "private",
        port: 5000
    ], 
    [
        name: 'module3',
        root: "${myRoot}",
        visibility: "private",
        port: 3000
    ]
]

def commonNodeFunction() {
    println("Installing Curl!")
    sh 'apk add --update curl'
    sh 'node --version'
    sh 'npm i'
    sh 'npm run test'
}

def displayCurrentDir() {
    println("Current Dir!")
    sh 'pwd'
    sh 'ls'
}

pipeline {
    environment {
        registry="jenkins-demo"
    }

    agent any

    stages {
        stage('Stage1') {
            steps {
                script {
                    node {
                        //ws {
                            displayCurrentDir();
                            sh "echo This comes from Stage1 ROOT > demo-file0.txt"

                            docker.image("node:13-alpine").inside("-u root:root") {
                                displayCurrentDir();
                                sh 'cat demo-file0.txt'
    							sh "echo This comes from docker 1 output > demo-file1.txt"
    						}

    						docker.image("node:12-alpine").inside("-u root:root") {
                                displayCurrentDir();
    							sh 'cat demo-file0.txt'
    							sh 'cat demo-file1.txt'
    							sh "echo This comes from docker 2 output > demo-file2.txt"
    						}

                            displayCurrentDir();

                            sh 'cat demo-file0.txt'
                            sh 'cat demo-file1.txt'
                            sh 'cat demo-file2.txt'
                            sh "echo FROM node:13-alpine > Dockerfile"
                            sh "echo RUN apk add --update nano > Dockerfile"

                            displayCurrentDir();

                            docker.image("python:2-alpine").inside("-u root:root") {
                                sh 'python --version'
    						}

                            docker.build registry + ":$BUILD_NUMBER"

                            docker.image("python:3-alpine").inside("-u root:root") {
                                sh 'python --version'
    						}
                        //}
                    }
                }
            }
        }

        stage('Stage2') {
            steps {
                script {
                    node {
                            displayCurrentDir();
                            sh 'cat demo-file0.txt'
                            sh 'cat demo-file1.txt'
                            sh 'cat demo-file2.txt'
                    }
                }
            }
        }

        stage('Stage3') {
            steps {
                script {
                    node {
                        checkout scm
                        def builders = [:]
                        for (x in modules) {
                            def module = x // Need to bind the label variable before the closure - can't do 'for (label in labels)'

                            // Create a map to pass in to the 'parallel' step so we can fire all the builds at once
                            builders["Stage3: " + module.name] = {
                                displayCurrentDir();

                                dir(module.name) {
                                    displayCurrentDir();
                                    echo "Name: ${module.name} Root: ${module.root} Port: ${module.port} Visibility: ${module.visibility}"
                                    sh "echo This comes from ${module.name} Stage3 > ${module.name}_demo-file0.txt"

                                    docker.image("node:13-alpine").inside("-u root:root") {
                                        displayCurrentDir();
                                        commonNodeFunction();

                                        sh "cat ${module.name}_demo-file0.txt"
            							sh "echo This comes from ${module.name} docker output 1 > ${module.name}_demo-file1.txt"
            						}

            						docker.image("node:12-alpine").inside("-u root:root") {
                                        displayCurrentDir();
                                        commonNodeFunction();

                                        sh "cat ${module.name}_demo-file0.txt"
            							sh "cat ${module.name}_demo-file1.txt"
            							sh "echo This comes from ${module.name} docker output 2 > ${module.name}_demo-file2.txt"
            						}

                                    docker.image("python:3-alpine").inside("-u root:root") {
                                        displayCurrentDir();

                                        sh "cat ${module.name}_demo-file0.txt"
            							sh "cat ${module.name}_demo-file1.txt"
                                        sh "cat ${module.name}_demo-file2.txt"
            							sh "echo This comes from ${module.name} docker output 3 > ${module.name}_demo-file3.txt"
            						}

                                    displayCurrentDir();

                                    sh "cat ${module.name}_demo-file0.txt"
                                    sh "cat ${module.name}_demo-file1.txt"
                                    sh "cat ${module.name}_demo-file2.txt"
                                    sh "cat ${module.name}_demo-file3.txt"

                                    docker.build "${module.name}:$BUILD_NUMBER"
                                }
                            }
                        }

                        parallel builders
                    }
                }
            }
        }
    }
}
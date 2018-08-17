## [Big Android BBQ 2015](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc_HyE1QX9heAgTPdAMqc50z)
 - [Internal Library Dependency Management](https://www.youtube.com/watch?v=EsOTPNAaH6o&index=6&list=PLNBb8OktVDKujbHIudyFzvLOmKQTFjCAs)
    - Single Repository , Single Gradle Module
        - Pros
            - Easy to understand
            - Everything in central location
            - Straightforward to  obtain code & debug
        - Cons
            - Lib is not separate component
                - Can't be shared w/ another team
                - Can't be versioned independently
            - No explicit separation of concerns.
    - Single Repository , Multiple Gradle Module
        - Pros
            - Easy procedure; hard to mess up
            - Add some seperation of concern
        - Cons
         - Lib is not separate component
            - Can't be shared w/ another team
            - Can't be versioned independently
    - Multiple Repository , Linked by Git(Git Submodules)
        - Implementation
            - Create Submodule-Sample(Main) repo and  Submodule-Lib(lib)
            - Add library inside Submodule-Lib named mylibrary.
            - In Submodule-Sample do following
            - git submodule add <lib-git-path
            ![ScreenShot](/screenshots/submodule/submodule-add.png)
            - git status
             ![ScreenShot](/screenshots/submodule/git-status.png)
            - Add Gradle References
                - Inside settings.gradle
                ```
                include ':mylibrary'
                project(':mylibrary').projectDir = new File('../Submodule-Lib/mylibrary')
                or
                include ':Submodule-Lib:mylibrary'
                ```
                - In build.gradle(app)
                ```
                compile project(':Submodule-Lib:mylibrary')
                ```
            - Check git submodule status
            ```
            git submodule
            ```
            - How to pull code down with submodule
                - Checkout new project
                ```
                git clone <repo> --recursive
                ```
                it do the following
                ```
                git clone <repo>
                git submodule init
                git submodule update
                ```
                - Checkout branch
                ```
                git checkout <branch>
                git submodule init
                git submodule update
                ```
            - Pros
                - Clear separation of concern
                - Library is in separate repo! Can be shared
                - Vesrion brought by git
                - Vesrion determined by git
                - One Android Project
                - Easy debug
                - Submodule maintain its own seprate git history
            - Cons
                - Magic is involved
                - Very easy to mess up
                    - Working with detach head
                    - Pulling down latest submodule
                    - Adding & Tracking submodule refrence
                - Multi step PR and merge
                - Library vesrions are commit SHAs

    - Multiple Repository , Linked by Maven

        - Setting Maven repository via [Artifactory](https://www.jfrog.com/open-source/)
            - Install Java 8
            - Download / unzip artifactory
            - Run bin/artifact.sh with `sudo ./artifact.sh`
            - Check http://localhost:8081/artifactory/webapp/#/home

        - CONFIGURING GRADLE TO UPLOAD ANDROID ARTIFACTS
            - build.gradle(project level)
                ```
                 classpath "org.jfrog.buildinfo:build-info-extractor-gradle:3.1.1"
                ```
            - build.gradle(lib level)

            ```
            apply plugin: 'com.android.library'
            apply plugin: 'com.jfrog.artifactory'
            apply plugin: 'maven-publish'

            def packageName = 'your package name'
            def libraryVersion = '1.0.0'
            ```

            ```
             publishing {
                publications {
                    aar(MavenPublication) {
                        groupId packageName
                        version = libraryVersion
                        artifactId project.getName()

                        // Tell maven to prepare the generated "*.aar" file for publishing
                        artifact("$buildDir/outputs/aar/${project.getName()}-release.aar")
                    }
                }
            }

            artifactory {
                contextUrl = 'http://localhost:8081/artifactory'
                publish {
                    repository {
                        // The Artifactory repository key to publish to
                        repoKey = 'libs-release-local'

                        username = "admin"
                        password = "password"
                    }
                    defaults {
                        // Tell the Artifactory Plugin which artifacts should be published to Artifactory.
                        publications('aar')
                        publishArtifacts = true

                        // Properties to be attached to the published artifacts.
                        properties = ['qa.level': 'basic', 'dev.team': 'core']
                        // Publish generated POM files to Artifactory (true by default)
                        publishPom = true
                    }
                }
            }
            ```

        - DEPLOYING ARTIFACTS
            ```
            ./gradlew assembleRelease artifactoryPublish
            ```
            - Check .aar file in http://localhost:8081/artifactory/webapp/#/artifacts/browse/tree/General/libs-release-local/

        - USING THE ARTIFACTS
            - Shell Repo(Sample)
                - build.gradle(project level)
                ```
                 allprojects {
                     repositories {
                         jcenter()
                         maven { url "http://localhost:8081/artifactory/libs-release-local" }
                     }
                 }
                ```
                - build.gradle(app level)
                ```
                compile 'com.jeroenmols.awesomelibrary:awesomelibrary:1.0.0'
                ```
            - Pulling Changes
            ```
             Preferences>Compiler>Command-line Options: --refresh-dependencies (this will run every time slowing build)
            ```

             or from command line
            ```
            ./gradlew build --refresh-dependencies
            ```

        - Debugging lib
          - Source Jar
        - How to use server insted of localhost (How to deploy jFrog on server)
        - Credentials
        - Setting up snapshots




        - http://jeroenmols.com/blog/2015/08/06/artifactory/
        - https://medium.com/@Mul0w/publish-with-gradle-on-bitbucket-1463236dc460#.h1um1rdg1
        - https://chris.banes.me/2013/08/27/pushing-aars-to-maven-central/


## Ref
 - [Codelabs](https://codelabs.developers.google.com/babbq-2015)

pipeline {
    agent any
    parameters {
        string(name: 'version', defaultValue: '2.346.1.1', description: 'What version of CloudBees CI do you want to check plugin tiers for?')
        string(name: 'envelope', defaultValue: 'envelope-core-cm', description: 'envelope-core-cm or envelope-core-oc')
        text(name: 'pluginsToCheck', defaultValue: '', description: '')
        booleanParam(name: 'groupResults', defaultValue: false, description: '')
    }
    stages {
        stage('Generate Report') { 
            steps {
                script {
                    //import groovy.json.JsonSlurper

                    // Initialise variables

                    def updateCenterURL = "https://jenkins-updates.cloudbees.com/update-center/${params.envelope}/update-center.json?version=${params.version}"
                    def groupResults = "${params.groupResults}"
                    def matchedVerified = []
                    def matchedCompatible = []
                    def notMatched = []
                    def nonGrouped = []
                    def nonGroupedOutputString = ""

                    // Download updatecenter.json
                    def get = new URL(updateCenterURL).openConnection(); 

                    def postResponseCode = get.getResponseCode();
                    def postResponseData = get.getInputStream().getText();

                    if (postResponseCode != 200) {
                        echo "update-center.json could not be retrieved from URL " + updateCenterURL
                        assert false
                    }

                    // Extract only the json line from the response
                    def lines = postResponseData.readLines()
                    def extractedResponseData = lines[1]

                    // Convert to Json
                    //def responseAsJson = jsonSlurper.parseText(extractedResponseData)
                    def slurper = new groovy.json.JsonSlurperClassic()
                    def responseAsJson = slurper.parseText(extractedResponseData)

                    // Get list of plugins to check and iterate through them
                    def pluginsToCheck = "${params.pluginsToCheck}"

                    def totalPluginsNumber = pluginsToCheck.readLines().size()

                    pluginsToCheck.readLines().each { plugin ->
                        // On each iteration extract the plugin id
                        def pluginId = plugin.split(":")

                        // try to match the plugin in the Json
                        def matched = responseAsJson.offeredEnvelope.plugins[pluginId[0]]
                        
                        // if plugin not matched
                        if (matched == null)
                        {
                            // if results should be grouped then record this plugin in it's correct group
                            if (groupResults == 'true') {
                                // record it in correct group
                                notMatched << pluginId[0]
                            } else {
                                // or record it in original order
                                nonGrouped << pluginId[0]
                            }
                        } else {
                            // if plugin is matched
                            // if plugin is verified or proprietary
                            if (matched.tier in ['verified', 'proprietary']) {
                                // if version was supplied then check to see if version matches
                                if (pluginId.size() > 1 && matched.version != pluginId[1]) {
                                    // record with version warning
                                    matchedVerified << matched.artifactId + " (CAP version " + matched.version + " supplied plugin version " + pluginId[1] + ")"
                                } else {
                                    // record without version warning
                                    matchedVerified << matched.artifactId
                                }
                            // if plugin is compatible
                            } else if (matched.tier == 'compatible') {
                                // if version was supplied then check to see if version matches
                                if (pluginId.size() > 1 && matched.version != pluginId[1]) {
                                    // record with version warning
                                    matchedCompatible << matched.artifactId + " (CAP version " + matched.version + " supplied plugin version  " + pluginId[1] + ")"
                                } else {
                                    // record without version warning
                                    matchedCompatible << matched.artifactId
                                }
                            }
                            // if results should not be grouped then display this plugin
                            if (groupResults == 'false') {
                                // if version was supplied then check to see if version matches
                                if (pluginId.size() > 1 && matched.version != pluginId[1]) {
                                    // record with version warning
                                    nonGrouped << matched.artifactId + ' | ' + matched.tier + " (CAP version " + matched.version + " supplied plugin version " + pluginId[1] + ")"
                                } else {
                                    // record without version warning
                                    nonGrouped << matched.artifactId + ' | ' + matched.tier
                                }
                            }
                        }
                    }

                    // if results should be grouped then display them
                    if (groupResults == 'true') {
                        echo "Verified or Proprietary\n--------\n"
                        matchedVerified.each { echo it }

                        echo "\nCompatible\n----------\n"
                        matchedCompatible.each { echo it }

                        echo "\nTier 3\n------\n"
                        notMatched.each { echo it }
                    } else {
                        nonGrouped.each { item ->
                            nonGroupedOutputString = nonGroupedOutputString + item + "\n"
                        }
                        echo "Combined string: " + nonGroupedOutputString
                    }

                    percentMatchedVerified = (matchedVerified.size() / totalPluginsNumber)*100
                    percentMatchedCompatible = (matchedCompatible.size() / totalPluginsNumber)*100
                    percentNonMatched = (notMatched.size() / totalPluginsNumber)*100

                    // Display some statistics
                    echo "\nStatistics\n"
                    echo "Total number of plugins = " + totalPluginsNumber
                    echo "Number of verified or proprietary plugins = " + matchedVerified.size() + " (" + percentMatchedVerified + "%)"
                    echo "Number of compatible plugins = " + matchedCompatible.size() + " (" + percentMatchedCompatible + "%)"
                    echo "Number of Tier 3 plugins = " + notMatched.size() + " (" + percentNonMatched + "%)"
                }
            }
        }
    }
}

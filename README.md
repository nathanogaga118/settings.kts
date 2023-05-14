# settings.kts

import jetbrains.buildServer.configs.kotlin.*
import jetbrains.buildServer.configs.kotlin.buildSteps.ScriptBuildStep
import jetbrains.buildServer.configs.kotlin.buildSteps.script

version = "2022.04"

project {
    description = "Contains all other projects"
    buildType(DastardlyScan)
}
object DastardlyScan : BuildType({
    name = "Dastardly from Burp Suite Scan"
    vcs {
        cleanCheckout = true
    }
    features {
        feature {
            type = "xml-report-plugin"
            param("xmlReportParsing.reportType", "junit")
            param("xmlReportParsing.reportDirs", "+:**/reports/**.xml")
        }
    }
    steps {
        script {
            name = "Dastardly from Burp Suite Scan"
            dockerImage = "public.ecr.aws/portswigger/dastardly:latest"
            dockerPull = true
            dockerImagePlatform = ScriptBuildStep.ImagePlatform.Linux
            dockerRunParameters = """
                -e DASTARDLY_TARGET_URL=https://ginandjuice.shop/
                -e DASTARDLY_OUTPUT_FILE=%teamcity.build.checkoutDir%/reports/dastardly-report.xml
            """.trimIndent()
            scriptContent = """
                mkdir -p %system.teamcity.build.workingDir%/reports
                docker-entrypoint.sh dastardly
            """.trimIndent()
        }

    }
    artifactRules = """
        reports/** => reports
    """.trimIndent()
})

# maven-settings-decoder
===========================================================================

A tool to decrypt the XML node text stored in maven settings.xml files

Inspired by [jelmerk/maven-settings-decoder](https://github.com/jelmerk/maven-settings-decoder)

Maven 2.1.0+  supports [server password encryption](http://maven.apache.org/guides/mini/guide-encryption.html)

This tool lets you decrypt these encrypted text as long as you have access to both the settings.xml file and the 
settings-security.xml file.

    usage: maven-settings-decoder
     -s,--settings <arg>             location of settings.xml file.
     -ss,--settings-security <arg>   location of settings-security.xml.
     -x,--xml-path <arg>             xpath expression, only node text is supported.

    or set '-s' and '-ss' in MAVEN_SETTINGS environment variable.

Use as a command-line tool:

    java -jar maven-settings-decoder-cli/target/maven-settings-decoder-cli-*-exec.jar \
        -s "${HOME}/.m2/settings.xml" \
        -ss "${HOME}/.m2/settings-security.xml" \
        -x "/settings/servers/server[id='private-nexus3-releases']/password/text()"
    
    # or
    
    java -jar maven-settings-decoder-cli/target/maven-settings-decoder-cli-*-exec.jar \
        -s "${HOME}/.m2/settings.xml" \
        -ss "${HOME}/.m2/settings-security.xml" \
        -x "//server[id='private-nexus3-releases']/password/text()"

Use as gradle buildscript dependency, so we can access maven's settings.xml from our build script:

        buildscript {
          // cn.home1.tools:maven-settings-decoder is in maven central.
          mavenCentral()
          // use sonatype-snapshots to get cn.home1.tools:maven-settings-decoder snapshot versions
          //maven { url 'https://oss.sonatype.org/content/repositories/snapshots/' }
          dependencies {
            ...
            classpath 'cn.home1.tools:maven-settings-decoder-cli:1.1.0'
          }
        }
        ...
        ext.mavenSettings = new cn.home1.tools.maven.SettingsDecoder();
        ext.nexusSnapshotsUser = mavenSettings.getText("//server[id='${nexus}-snapshots']/username/text()")
        ext.nexusSnapshotsPass = mavenSettings.getText("//server[id='${nexus}-snapshots']/password/text()")
        println "${nexus}-snapshots username: " + mavenSettings.getText("//server[id='${nexus}-snapshots']/username/text()")
        println "${nexus}-snapshots password: " + mavenSettings.getText("//server[id='${nexus}-snapshots']/password/text()")
        ...

### Build this package

```bash
./mvnw -s settings.xml clean install
CI_OPT_OSSRH_NEXUS2_USER=user CI_OPT_OSSRH_NEXUS2_PASS=password ./mvnw -Dgpg.executable=gpg -Dgpg.loopback=true -s settings.xml clean deploy
```

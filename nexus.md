# Nexus Repository Manager –

learning order:

1. Nexus introduction.
2. GitHub vs Nexus.
3. DevOps flow diagram.
4. Nexus installation on AWS EC2.
5. Troubleshooting.
6. Nexus configuration.
7. Repository creation.
8. Maven integration.
9. Artifact deployment.
10. Snapshot vs Release.
11. Real-time interview questions.

The explanations are kept simple, with commands and real-time use cases.

## 1. Introduction to Nexus

What is Nexus?

- Nexus is a repository manager that serves as a central hub for managing binary artifacts such as libraries, Docker images, and other build artifacts.
- It provides a secure and efficient way to store, distribute, and manage software components and dependencies throughout the software development lifecycle.
- It is an open-source software product.
- It is a Java-based repository manager, meaning it is developed using Java.

Vendor: Sonatype.

Product name: Sonatype Nexus Repository.

## 2. Types of Repository Managers

1. Nexus – Developed by Sonatype.
2. JFrog Artifactory – Developed by JFrog.

## 3. Difference Between GitHub and Nexus

| GitHub                                   | Nexus                                               |
| ---------------------------------------- | --------------------------------------------------- |
| Used to manage source code.              | Used to manage build artifacts.                     |
| Stores application code and Dockerfiles. | Stores WAR, JAR, Docker images and other artifacts. |
| Developers push source code.             | Build tools upload generated artifacts.             |
| Used for source code management.         | Used for artifact management.                       |

Interview Question: What is the difference between GitHub and Nexus?

Answer:

- GitHub is used to manage source code and Dockerfiles.
- Nexus is used to manage build artifacts and Docker images.

## 4. Explain the DevOps Flow Diagram

## CI/CD Flow – Developer to Tomcat

1. Developer (Dev1)

   Developer writes application code and pushes it to GitHub.
2. GitHub

   Stores source code and manages version control.
3. Maven

   Downloads dependencies, compiles code, runs tests and generates a WAR file.
4. Nexus (Artifact Repository)

   Stores and manages the generated WAR file for future use and deployment.
5. Tomcat

   Deploys and runs the WAR file as a web application.

### Flow Diagram

```
Developer
    |
    v
  GitHub
    |
    v
  Maven
    |
    | Generates WAR file
    v
  Nexus
    |
    | Downloads WAR file
    v
  Tomcat
    |
    v
Application Running
```

### Explanation of Each Component

1\. Developer

- Developer writes application code.
- Pushes the code to GitHub.

2\. GitHub

- Stores application source code.
- Maintains version history.
- Developers can collaborate through branches and pull requests.

3\. Maven

- Maven is a build automation tool.
- Downloads dependencies.
- Compiles source code.
- Runs test cases.
- Generates the WAR file.

4\. Nexus

- Stores the generated WAR file.
- Maintains different artifact versions.
- Provides a central repository for build artifacts.

5\. Tomcat

- Tomcat is a Java web application server.
- Takes the WAR file.
- Deploys and runs the application.

Real-time use case:

Whenever a developer changes application code and pushes it to GitHub, the CI/CD pipeline can build a new WAR file, upload it to Nexus, and deploy the artifact to Tomcat.

## 5. Nexus Installation on AWS EC2

### 5.1 Prerequisites

Software Requirements

| Component           | Requirement in original lab              |
| ------------------- | ---------------------------------------- |
| Java                | Java 8 for Nexus 3.x Java 8 distribution |
| Operating System    | Linux                                    |
| Architecture        | Cross-platform software                  |
| Installation format | Archive file (TAR.GZ)                    |

Hardware Requirements

- Minimum 2 GB RAM for Nexus.
- Additional 1 GB RAM for the server, according to the original lab.
- We are going with `t2.medium`.

Important: Java compatibility depends on the exact Nexus version. The Java 8 distribution used below is an older Nexus release, not a universal requirement for every Nexus 3.x release.

### 5.2 Important Directories Created After Installation

After extracting and starting Nexus, two directories are used:

1. `nexus-3.x` – Contains Nexus application files.
2. `sonatype-work` – Contains Nexus data, configuration and repository storage.

Note: `sonatype-work` is used for persistent Nexus data. It is not a common directory shared by every installed Nexus version.

## 6. Nexus Installation – Step by Step

### Step 1: Launch an EC2 Instance

Launch an EC2 instance:

```
Instance type: t2.medium
Operating System: Linux
```

### Step 2: Connect to the Server

Connect to your EC2 instance using SSH or Git Bash.

### Step 3: Check RAM

Before installation, check the RAM:

```
free -h
```

This displays the total, used and available memory.

### Step 4: Switch to Root User

```
sudo su -
```

### Step 5: Check Whether Java Is Installed

```
javac -version
```

If Java is not installed, proceed with the next step.

### Step 6: Install Java 8 (Amazon Corretto)

Import the Corretto key:

```
rpm --import https://yum.corretto.aws/corretto.key
```

Download the repository configuration:

```
curl -Lo /etc/yum.repos.d/corretto.repo https://yum.corretto.aws/corretto.repo
```

Install Java 8 development package:

```
yum install -y java-1.8.0-amazon-corretto-devel --nogpgcheck
```

Verify:

```
java -version
javac -version
```

### Step 7: Navigate to /opt and Install Required Packages

```
cd /opt
```

Install required packages:

```
yum install tar wget tree -y
```

Purpose:

- `tar` – Extracts archive files.
- `wget` – Downloads files from the internet.
- `tree` – Displays directory structures.

### Step 8: Download Nexus

```
wget https://download.sonatype.com/nexus/3/nexus-3.70.1-02-java8-unix.tar.gz
```

### Step 9: Extract Nexus

```
tar -zxvf nexus-3.70.1-02-java8-unix.tar.gz
```

### Step 10: Rename Nexus Directory

```
mv /opt/nexus-3.70.1-02 /opt/nexus
```

Now the directory structure will look like:

```
/opt/
    |
    |-- nexus/
    |
    |-- sonatype-work/
```

## 7. Create a Dedicated Nexus User

Important Note:

As a good security practice, Nexus should not run as the root user.

Instead, create a separate user called `nexus` and give that user ownership of the Nexus directories.

### Step 11: Create Nexus User

```
useradd nexus
```

### Step 12: Give Sudo Access to Nexus User

Open sudoers file:

```
visudo
```

Add the following line:

```
nexus ALL=(ALL) NOPASSWD: ALL
```

Save and exit.

Purpose: Allows the Nexus user to execute sudo commands without entering a password.

### Step 13: Change Ownership and Permissions

Change ownership of Nexus directory:

```
chown -R nexus:nexus /opt/nexus
```

Change ownership of Sonatype work directory:

```
chown -R nexus:nexus /opt/sonatype-work
```

Set permissions:

```
chmod -R 775 /opt/nexus

chmod -R 775 /opt/sonatype-work
```

Explanation:

- `chown` – Changes ownership.
- `nexus:nexus` – User and group.
- `chmod 775` – Owner and group get full permissions; others get read and execute permissions.

### Step 14: Configure Nexus to Run as Nexus User

Open the Nexus configuration file:

```
vi /opt/nexus/bin/nexus.rc
```

Find:

```
#run_as_user=""
```

Uncomment and update:

```
run_as_user="nexus"
```

Save and exit.

### Step 15: Create Nexus as a Service

The original mentor notes include this symbolic link command:

```
ln -s /opt/nexus/bin/nexus /etc/init.d/nexus
```

This is an older SysV-style service approach.

For a systemd-based EC2 Linux server, use the following service configuration instead.

Create the service file:

```
sudo vi /etc/systemd/system/nexus.service
```

Add:

```
[Unit]
Description=Nexus Repository Manager
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

Save and exit.

### Step 16: Reload Systemd

```
sudo systemctl daemon-reload
```

### Step 17: Enable Nexus Service

```
sudo systemctl enable nexus
```

### Step 18: Start Nexus Service

```
sudo systemctl start nexus
```

Check status:

```
sudo systemctl status nexus
```

Switch to Nexus user if you need to perform Nexus user-level operations:

```
sudo su - nexus
```

### Step 19: Access Nexus from Laptop/Desktop

Open the browser:

```
http://<EC2-PUBLIC-IP>:8081/
```

Example from the original lab:

```
http://52.66.73.219:8081/
```

Important: Open port `8081` in the EC2 Security Group to access Nexus.

### Step 20: Login to Nexus

Click on Login.

Note: Before Nexus 3.15, the original lab mentions the username and password as:

```
Username: admin
Password: admin123
```

In newer Nexus versions, the initial admin password is generated automatically.

Check the initial password:

```
cat /opt/sonatype-work/nexus3/admin.password
```

Login:

```
Username: admin
Password: <password from admin.password file>
```

After login, follow the prompts to set a new admin password.

## 8. Nexus Troubleshooting

### Problem 1: Nexus Service Is Not Starting

Check the following:

A. Check ownership and permissions

```
ls -ld /opt/nexus

ls -ld /opt/sonatype-work
```

If ownership is incorrect:

```
chown -R nexus:nexus /opt/nexus

chown -R nexus:nexus /opt/sonatype-work
```

Set permissions:

```
chmod -R 775 /opt/nexus

chmod -R 775 /opt/sonatype-work
```

B. Make sure Nexus runs with the Nexus user

```
su - nexus
```

C. Check Java installation

```
java -version
```

D. Check Nexus logs

Navigate to the log directory:

```
cd /opt/sonatype-work/nexus3/log
```

Check the log:

```
tail -n 100 nexus.log
```

E. Check systemd service logs

```
sudo journalctl -u nexus -n 100 --no-pager
```

### Problem 2: Unable to Access Nexus URL

Check:

1. Make sure Nexus service is running.
2. Verify that port `8081` is opened in the AWS EC2 Security Group.
3. Verify that you are using the correct public IP.
4. Check whether the application is listening on port 8081.

Commands:

```
sudo systemctl status nexus
```

```
ss -lntp | grep 8081
```

Access:

```
http://<EC2-PUBLIC-IP>:8081/
```

## 9. How to Change Port Number and Context Path?

Interview Question: How do you change the default port number of Nexus?

### Step 1: Navigate to Nexus Configuration Directory

```
cd /opt/nexus/etc/
```

### Step 2: Open Configuration File

```
vi nexus-default.properties
```

Find the application configuration:

```
application-port=8081
```

Change the port number as required.

Example:

```
application-port=8082
```

Save and exit.

### Step 3: Restart Nexus

```
sudo systemctl restart nexus
```

### Step 4: Open the New Port in AWS

Make sure the new port is enabled in the EC2 Security Group.

Access:

```
http://<EC2-PUBLIC-IP>:8082/
```

Note: The context path can also be configured in the same properties file, depending on the Nexus version.

## 10. How to Disable Anonymous Access?

Problem: Before logging into the Nexus server, browsing options are available.

Requirement: We want to close browsing access for users who are not logged in.

### Steps

1. Login as root/admin user.
2. Go to Settings.
3. Navigate to Security.
4. Open Anonymous Access.
5. Disable the Anonymous Access option.
6. Save the changes.

Result: Users must log in before accessing repositories and browsing protected content.

## 11. How to Change the Nexus Admin Password?

### Steps

1. Login as admin.
2. Click on the admin profile.
3. Open the account or password settings.
4. Enter the required password details.
5. Save the changes.

## 12. How to Create a Repository in Nexus?

Requirement: For every project, we are going to create two types of repositories.

1. Snapshot Repository.
2. Release Repository.

### Step 1: Create Snapshot Repository

1. Login as admin.
2. Go to Settings.
3. Select Repositories.
4. Click Create Repository.
5. Select `maven2 (hosted)`.
6. Give the repository name:

```
jio-snapshot
```

7\. Select Version Policy:

```
Snapshot
```

8. Scroll down.
9. Click Create Repository.

### Step 2: Create Release Repository

1. Login as admin.
2. Go to Settings.
3. Select Repositories.
4. Click Create Repository.
5. Select `maven2 (hosted)`.
6. Give the repository name:

```
jio-release
```

7\. Select Version Policy:

```
Release
```

8. Scroll down.
9. Click Create Repository.

### Repository Structure

```
Nexus
  |
  |-- jio-snapshot
  |
  |-- jio-release
```

### Repository URLs

Each repository has a separate URL.

Release Repository:

```
http://<NEXUS-IP>:8081/repository/jio-release/
```

Snapshot Repository:

```
http://<NEXUS-IP>:8081/repository/jio-snapshot/
```

Example URLs from the original notes:

```
http://3.111.47.165:8081/repository/jio-release/
```

```
http://3.111.47.165:8081/repository/jio-snapshot/
```

Important Note:

For each project, we are creating two repositories:

- `jio-snapshot`
- `jio-release`

## 13. How to Integrate Nexus with Maven?

Requirement: We need to integrate Maven with Nexus so that Maven can upload generated artifacts into the Nexus repository.

### Step 1: Connect to Maven Server

Connect to the Maven EC2 instance where Java projects are available.

Navigate to the project directory:

```
cd /path/to/project
```

The project directory should contain:

```
pom.xml
src/
```

### Step 2: Where to Keep Nexus Repository Details?

We need to update the Nexus repository details in the project's `pom.xml`.

Open:

```
vi pom.xml
```

Add the following configuration inside the `<project>` element:

```
<distributionManagement>

    <repository>
        <id>nexus</id>
        <name>Releases Nexus Repository</name>
        <url>http://<NEXUS-IP>:8081/repository/jio-release/</url>
    </repository>

    <snapshotRepository>
        <id>nexus</id>
        <name>Snapshot Nexus Repository</name>
        <url>http://<NEXUS-IP>:8081/repository/jio-snapshot/</url>
    </snapshotRepository>

</distributionManagement>
```

Explanation:

- `<distributionManagement>` – Defines where Maven should upload artifacts.
- `<repository>` – Defines the release repository.
- `<snapshotRepository>` – Defines the snapshot repository.
- `<id>` – Repository identifier used to match credentials in Maven settings.
- `<url>` – Nexus repository URL.

### Step 3: Where to Keep Nexus Credentials?

Difference between SonarQube and Nexus:

- In the original SonarQube setup, credentials were configured in `pom.xml`.
- In Nexus, repository details are configured in `pom.xml`, but credentials are configured in Maven's `settings.xml`.

Navigate to Maven configuration:

```
cd /opt/apache-maven-3.9.10/conf/
```

Open:

```
vi settings.xml
```

Add the server configuration inside the existing `<servers>` section:

```
<servers>

    <server>
        <id>nexus</id>
        <username>admin</username>
        <password>YOUR_NEXUS_PASSWORD</password>
    </server>

</servers>
```

Important: The `<id>` in `settings.xml` must match the `<id>` in `pom.xml`.

For security, use a dedicated Nexus deployment account with only the necessary repository permissions rather than using the admin account.

## 14. How to Deploy Artifacts into Nexus?

Interview Question: How do you deploy artifacts into Nexus?

### Step 1: Navigate to Project Directory

Go to the project directory where `pom.xml` is available.

```
cd /path/to/maven-project
```

### Step 2: Execute Maven Deploy Command

```
mvn clean deploy
```

### What Happens When We Execute `mvn clean deploy`?

1. `clean` – Removes the previous build output.
2. `validate` – Validates the project.
3. `compile` – Compiles the source code.
4. `test` – Executes unit test cases.
5. `package` – Generates the WAR/JAR file.
6. `install` – Installs the artifact into the local Maven repository.
7. `deploy` – Uploads the artifact into the remote Nexus repository.

Note: If you execute `mvn clean deploy` multiple times with a SNAPSHOT version, Maven can upload updated timestamped snapshot artifacts into the Nexus snapshot repository.

## 15. Why Is Maven Always Uploading Artifacts into the Snapshot Repository?

Interview Question: Why is Maven always uploading artifacts into the snapshot repository instead of the release repository?

Answer:

The reason is that in `pom.xml`, we have defined the project version with `SNAPSHOT`.

Example:

```
<version>1.0.0-SNAPSHOT</version>
```

When the version contains `SNAPSHOT`, Maven uploads the artifact into the snapshot repository.

### How to Upload into Release Repository?

Remove `SNAPSHOT` from the version.

Update `pom.xml`:

```
<version>1.0.0</version>
```

Now execute:

```
mvn clean deploy
```

The artifact will be uploaded into the release repository.

### Difference Between Both

| Project Version  | Repository          |
| ---------------- | ------------------- |
| `1.0.0-SNAPSHOT` | Snapshot repository |
| `1.0.0`          | Release repository  |
| `2.0.0-SNAPSHOT` | Snapshot repository |
| `2.0.0`          | Release repository  |

### Important Note: Release Version Redeployment

By default, Nexus release repositories generally do not allow redeployment of an existing release version.

Example:

```
<version>1.0.0</version>
```

First deployment:

```
mvn clean deploy
```

BUILD SUCCESS

If you try to deploy the same release version again without changing the version, you may receive a Bad Request (400) error because redeployment is disabled.

Example:

```
400 Bad Request
```

### How to Allow Redeployment?

1. Login to Nexus as admin.
2. Go to Settings.
3. Select Repositories.
4. Open `jio-release`.
5. Scroll down.
6. Find Deployment Policy under Hosted.
7. Change the policy to `Allow redeploy`.
8. Save the changes.

Now execute:

```
mvn clean deploy
```

The same release version can be uploaded again.

Best Practice: After completing the required deployment, disable redeployment again. In production, release artifacts should normally be immutable.

## 16. Difference Between Snapshot and Release

| Snapshot                                            | Release                              |
| --------------------------------------------------- | ------------------------------------ |
| Used during continuous development and integration. | Used for stable, finalized versions. |
| Version contains `SNAPSHOT`.                        | Version does not contain `SNAPSHOT`. |
| Can be updated with newer snapshot builds.          | Normally immutable after deployment. |
| Uploaded to snapshot repository.                    | Uploaded to release repository.      |
| Used during development and testing.                | Used for production releases.        |

### Example

Snapshot Version:

```
<version>1.0.0-SNAPSHOT</version>
```

Release Version:

```
<version>1.0.0</version>
```

Interview Answer:

- Snapshot is used during continuous development and testing.
- Release is used when the application version is stable and ready for release or production deployment.

## 17. How to Store Artifacts in Maven Local Repository?

Interview Question: How do you store artifacts in the Maven local repository?

Command:

```
mvn clean install
```

Explanation:

- Maven builds the project.
- Generates the WAR/JAR file.
- Stores the artifact in the local Maven repository.

Default local repository location:

```
~/.m2/repository/
```

Example:

```
/root/.m2/repository/
```

## 18. How to Store Artifacts in Maven Remote Repository?

Interview Question: How do you store artifacts in the Maven remote repository?

Command:

```
mvn clean deploy
```

Explanation:

- Maven builds the project.
- Generates the WAR/JAR file.
- Installs it into the local repository.
- Uploads it into the configured remote Nexus repository.

## 19. Real-Time Scenario: Release 1.0.0 Has Bugs

Requirement:

Release `1.0.0` has some bugs. I want to redeploy the artifact with the same version.

Solution:

### Step 1: Login to Nexus

Login using the admin account.

### Step 2: Open Release Repository

Navigate:

```
Settings
    ↓
Repositories
    ↓
jio-release
```

### Step 3: Change Deployment Policy

Scroll down.

Find:

```
Deployment Policy
```

Change it to:

```
Allow redeploy
```

Save the changes.

### Step 4: Redeploy Using Maven

On the Maven server:

```
mvn clean deploy
```

Now Nexus allows the same release version to be uploaded again.

### Step 5: Disable Redeployment

After redeployment:

1. Go back to Nexus.
2. Open `jio-release`.
3. Change Deployment Policy back to `Disable redeploy`.
4. Save.

Important: Redeploying the same release version is generally discouraged in production because the same version can then point to different binary contents. Prefer creating a new release version, such as `1.0.1`.

# 20. Important Maven and Nexus Commands – Quick Revision

| Command             | Purpose                                         |
| ------------------- | ----------------------------------------------- |
| `mvn clean`         | Removes previous build output                   |
| `mvn compile`       | Compiles source code                            |
| `mvn test`          | Executes unit tests                             |
| `mvn package`       | Generates WAR/JAR                               |
| `mvn install`       | Stores artifacts in local Maven repository      |
| `mvn deploy`        | Uploads artifacts to remote repository          |
| `mvn clean install` | Cleans, builds and installs locally             |
| `mvn clean deploy`  | Cleans, builds and uploads to remote repository |

### Important Nexus Installation Commands

```
# Check RAM
free -h

# Check Java
java -version

# Check Java compiler
javac -version

# Navigate to opt
cd /opt

# Install required packages
yum install tar wget tree -y

# Download Nexus
wget https://download.sonatype.com/nexus/3/nexus-3.70.1-02-java8-unix.tar.gz

# Extract Nexus
tar -zxvf nexus-3.70.1-02-java8-unix.tar.gz

# Rename directory
mv /opt/nexus-3.70.1-02 /opt/nexus

# Create Nexus user
useradd nexus

# Change ownership
chown -R nexus:nexus /opt/nexus
chown -R nexus:nexus /opt/sonatype-work

# Set permissions
chmod -R 775 /opt/nexus
chmod -R 775 /opt/sonatype-work

# Configure Nexus user
vi /opt/nexus/bin/nexus.rc

# Start Nexus
sudo systemctl start nexus

# Check status
sudo systemctl status nexus

# Enable Nexus at boot
sudo systemctl enable nexus

# Restart Nexus
sudo systemctl restart nexus

# Stop Nexus
sudo systemctl stop nexus

# Check Nexus logs
tail -n 100 /opt/sonatype-work/nexus3/log/nexus.log

# Check initial admin password
cat /opt/sonatype-work/nexus3/admin.password
```

# 21. Important Nexus Interview Questions

Q1. What is Nexus?

Nexus is a repository manager used to store, manage and distribute binary artifacts such as WAR, JAR, Docker images and other build artifacts.

Q2. Who developed Nexus?

Sonatype.

Q3. What is the default Nexus port?

Q4. Which user should run Nexus?

A dedicated non-root user, such as `nexus`.

Q5. What are the two important directories created after Nexus installation?

- Nexus application directory.
- Sonatype work directory.

Q6. What is the difference between GitHub and Nexus?

GitHub manages source code, whereas Nexus manages build artifacts.

Q7. What is a Snapshot repository?

It stores development versions of artifacts that can be updated during development.

Q8. What is a Release repository?

It stores stable, versioned artifacts intended for release.

Q9. What is the difference between `mvn install` and `mvn deploy`?

- `mvn install` stores the artifact in the local Maven repository.
- `mvn deploy` uploads the artifact to the remote repository.

Q10. Where do we configure Nexus repository details in Maven?

In `pom.xml`, using the `distributionManagement` section.

Q11. Where do we configure Nexus credentials?

In Maven's `settings.xml`, inside the `servers` section.

Q12. Why do we create two repositories for a project?

To manage snapshot and release versions separately.

Q13. What happens when we execute `mvn clean deploy`?

Maven cleans the previous build, builds the application, runs tests, packages the artifact, installs it locally and uploads it to the configured remote repository.

Q14. Why is the artifact uploaded to the snapshot repository?

Because the project version in `pom.xml` contains `SNAPSHOT`.

Q15. Can we redeploy the same release version?

By default, Nexus release repositories prevent redeployment. We can change the deployment policy to Allow redeploy, but this is not recommended for production.

## 22. Final DevOps Flow – Maven + Nexus + Tomcat

## Real-Time CI/CD Architecture

Developer

Writes application code

GitHub

Source code management

Maven

Builds project and generates WAR/JAR

Nexus

Stores and manages artifacts

Tomcat

Deploys and runs the WAR application

Application Running

Remember these five components:

| Component | Main Responsibility              |
| --------- | -------------------------------- |
| Developer | Writes code                      |
| GitHub    | Manages source code              |
| Maven     | Builds the application           |
| Nexus     | Stores build artifacts           |
| Tomcat    | Deploys and runs the application |

One important clarification: Your original flow shows GitHub → Maven → Tomcat. Nexus is the additional artifact-management step. In a real pipeline, Maven uploads the WAR to Nexus, and a deployment stage retrieves the WAR from Nexus and deploys it to Tomcat.

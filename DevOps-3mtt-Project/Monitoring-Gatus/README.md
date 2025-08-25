# COMFIGURE UPTIME MONITORING USING GATUS

## Introduction

Ensuring that your service and websites are available and performing as expected is crucial for maintaining user satisfaction and trust. Gatus is a simple yet powerful tool for monitoring the uptime of services and websites.

This project guild me through setting up Gatus to monitor the availability of a website or API endpoint and receive alerts when it becomes unavailable.

## Objectives

1. Understand what Gatus is, its role in uptime monitoring
2. Set up Gatus on my local machine or server
3. Configure Gatus to monitor one or more endpoints.
4. Set up alerting for downtime events.
5. Visualize monitoring results through the Gatus dashboard.

## Prerequisites
1. **Basic Knowledge**: Familiarity with HTTP service, APIs, and configuration files (YAML).
2. **Tools Required:**
- A machine with Docker installed (recommended for ease of setup).
- A text editor for editing configuration files.
- Internet access for tesing live endpoints.

# Project Tasks

## Task 1 - Install and Set Up Gatus Locally

1. Install Docker if it's not already installed. I am using a docker server and Ubuntu terminal

![](./Images/1.%20docker.png)

![](./Images/1c.%20Docker%20Version.png)

2. Pull the Gatus Docker image from Dockerhub.

``` bash
docker pull twinproduction/gatus
```

![](./Images/2.%20docker-pull.png)

![](./Images/1a.%20docker.png)

3. Create a directory for Gatus configuration

``` bash
mkdir gatus && cd gatus
```

4. Start Gatus using Docker with a basic setup.

``` bash
docker run -d -p 8080:8080 --name gatus -v $(pwd)/config:/config twinproduction/gatus
```

![](./Images/3.%20Docker%20PS-A.png)

the docker container exited immediately because the configuration directory is empty.


5. Access the Gatus dashboard in my browser at http://localhost:8080. But is not going to work for now because there's no YAML script define for Gatus for now.

![](./Images/4.%20CURL.png)


Since I can't curl the link on my terminal there is not possible to view it on the browser.

# Task 2 - Create a Basic Configuration File

1. Inside the gatus/config directory create a new file config.yaml
2. Add a simple configuration to monitor a website.


``` bash
endpoints:
  - name: Example Website
    url: "https://google.com"
    interval: 60s
    conditions:
      - "[STATUS] == 200"
```

3. Restart the Gatus container to apply the configuration.

``` bash
docker restart gatus
```

![](./Images/5.%20Docker%20UP%20(1).png)


Now the docker container is running because I have added the configuration file.

4. Now I test the URL on my browser to see the outcome


![](./Images/6.%20Browser.png)


# Task 3 - Test the Setup with Another Live Endpoint

Add another endpoint to the config.yaml file for monitoring.

``` bash
cat >> config.yaml
```

``` bash
- name: GitHub
  url: "https://github.com"
  interval: 60s
  conditions:
    - "[STATUS] == 200"
```

2. Restart the Gatus container and verify the new endpoint appears on the dashboard.

``` bash
docker restart gatus
```

![](./Images/7.%20Config%20Added.png)


3. Then check the the content on the browser

![](./Images/8.%20GItHub%20Added.png)


4. Simulate a failure by adding a non-existent endpoint and observer the behavior.


``` bash
cat >> config.yaml
```

``` bash
- name: Nonexistent
  url: "https://thiswebsitedoesnotexist.com"
  interval: 60s
  conditions:
    - "[STATUS] == 200"
```

5. Then restart the Gatus container.

``` bash
docker restart gatus
```

![](./Images/9.%20Non%20Exist%20Website.png)


# Task 4 - Configure Alerts for Downtime

1. Choose an alerting method, such as Slack or email. For example. for Gmail:` olisamasandra@gmail.com`

2. Add an alert configuration to `config.yaml`.

``` bash
alerts:
  - type: slack
    url: "https://hooks.slack.com/services/Wf0980GJHPK6"
    failure-threshold: 2
    success-threshold: 2
```

![](./Images/10.%20Alert.png)


3. Test the alearting by taking down a monitored service temporarily.


# Task 5 - Explore and Customize the Gatus Dashboard

1. Access the Gatus dashboard to view uptime statistics for each endpoint.
2. Customize the dashboard appearance (e.g., themes, logos) by modifying the configuration file.
3. Adjust monitoring intervals and conditions to optimize performance.
4. Then I update my config.yaml file, by adding the following content.


``` bash
ui:
  title: "My Uptime Monitor"
  theme: "dark"  # Options: "dark", "light"
  logo: "https://upload.wikimedia.org/wikipedia/commons/a/ab/Logo_TV_2015.png"  # Optional logo URL
  link:
    - name: "GitHub"
      url: "https://github.com/twinproduction/gatus"
    - name: "My Company"
      url: "https://orisuniyanu.com"
```

5. Here's how my final configuration file look like

``` bash
ui:
  title: "My Uptime Monitor"
  theme: "dark"
  logo: "https://upload.wikimedia.org/wikipedia/commons/a/ab/Logo_TV_2015.png"
  links:
    - name: "GitHub"
      url: "https://github.com/twinproduction/gatus"
    - name: "My Company"
      url: "https://orisuniyanu.com"

endpoints:
  - name: Example Website
    url: "https://google.com"
    interval: 60s
    conditions:
      - "[STATUS] == 200"
    alerts:
      - type: slack
        url: "https://hooks.slack.com/services/YOUR/REAL/WEBHOOK"
        failure-threshold: 2
        success-threshold: 2

  - name: GitHub
    url: "https://github.com"
    interval: 60s
    conditions:
      - "[STATUS] == 200"
    alerts:
      - type: slack
        url: "https://hooks.slack.com/services/YOUR/REAL/WEBHOOK"
        failure-threshold: 2
        success-threshold: 2

  - name: Nonexistent
    url: "https://thiswebsitedoesnotexist.com"
    interval: 60s
    conditions:
      - "[STATUS] == 200"
    alerts:
      - type: slack
        url: "https://hooks.slack.com/services/YOUR/REAL/WEBHOOK"
        failure-threshold: 2
        success-threshold: 2
```

![](./Images/11.%20Final%20Browser.png)


# Conclusion

In this project, I learnt how to se up and configure Gatus for monitoring uptime and performance of service and websites. You explored essential features like endpoint monitoring, alerting anf dashboard visualization. With this knowledge, I can expend the configuration to monitor multiple service, integrate with advanced alerting tools, and deploy Gatus in production environments.






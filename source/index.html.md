---
title: API Reference

language_tabs:
  - shell
  - ruby
  - python

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Lab Automated API Documentation. Lab Automated allows faster, more reliable static and dynamic security testing for mobile apps. Ongoing updates with our API will be kept up to date as functionality is released.

If you would like to trial Lab Automated, please go to https://lab.nowsecure.com and request access to our closed beta. You will be contacted shortly after by one of our Sales Representatives to set up a trial.

## Authorization

Request and response bodies are json encoded.

All requests require an OAuth access token. You can find your access token by going to https://lab-beta.nowsecure.com/account. Authorization tokens are currently authorized for 24 hours.

All requests should include:

`Authorization: Bearer {NS_TOKEN}`

## API Host

The Lab Automated API is hosted at:

https://lab-api.nowsecure.com

# Account

An `Account` represents a Lab Automated account. Multiple accounts can be associated with a single instance of Lab Automated all with the ability to run assessments or view application results.

## View Account Details

You can view account details for Lab Automated using the `GET /account` path.

    ```shell
    curl -H "Authorization: Bearer $NS_TOKEN" -X GET $HOST/account
    ```


The response will contain the period in which your account will be active along with the number of applications and assessments left on your account.

```json
    {
      "period": 172800,
      "limits": {
        "assessments": 2
      },
      "id": "363df3a1-0e8b-4960-bb6e-cc09aced37a1"
    }
```


## List of Current Account Users

You can view users that are associated with an account using `GET /account/user/`.

```shell
    curl -H "Authorization: Bearer $NS_TOKEN" -X GET $HOST/account/user/
```

The response will contain the `user id`, `email`, and permissions for the user under `scopes`. The `account id` will also be returned. Each user will be associated with the same `account`

```json

    {
      "id": "66a46874-eab4-4871-83c9-3b586968547c",
      "email": "user@nowsecure.com",
      "scopes": [
        "user:login"
      ],
      "account": "363df3a1-0e8b-4960-bb6e-cc09aced37a1"
    }
```

## View User Profile

You can view details about a specific user. Using `GET /account/user/{id}`, you can view that users `name`, `email`, `scope` also known as user permissions, and which `account` they are associated with.


```shell
  curl -H "Authorization: Bearer $NS_TOKEN" -X GET $HOST/account/user/27a03ce7-303f-440f-b02a-de2df2c41561
```

The .json response will include `name`, `email`, `scope`, and associated `account`

```json
    {
     "id": "27a03ce7-303f-440f-b02a-de2df2c41561",
     "name": "Charlie Painchaud",
     "email": "example@nowsecure.com",
     "scopes": [
       "user:login"
     ],
     "account": "60b923d9-7d14-4208-9630-f13075a58ffd"
    }
```

## Inviting a User

You can invite a user to join your Lab Automated account using `POST /account/invite/`. Users currently will have the same scope and will upload apps and run assessments against the `Account` limits.

## Pending Invites

You can view a list of pending invites using `GET /account/invite/`. This will return the `name` and `email` of each user that has been invited to the `account`

## Removing a User

Conversely, you can remove a user from your account using `DELETE /account/user/{id}`. The user `id` is returned in the `GET /account/user/` response.

# Applications

An `Application` or a binary is what Lab Automated will be performing the NowSecure Security Assessment against. An `Application` is a fully compiled .apk or .ipa file that will be installed onto real devices and security tested statically and dynamically.

## Submitting an Application

Using `POST /build/` will create a new application and kick off an assessment with the previously set configuration options. If it is the first time an application has been uploaded, the assessment will start with the default configuration options.

```shell
curl -H "Authorization: Bearer $NS_TOKEN" -X POST $HOST/build/ --data-binary @bin.apk
```

##  View all Applications

You can list all of the applications associated with an account using `GET /app/`.

```shell
    curl -H "Authorization: Bearer $NS_TOKEN" -X GET $HOST/app/
```

For each `Application` the response will contain the apps `platform`, `package` name, associated `account`, and `created` date.

```json
    {
    "platform": "android",
    "package": "fuzion24.dynamictestapp",
    "account": "363df3a1-0e8b-4960-bb6e-cc09aced37a1",
    "created": "2016-04-07T20:29:15.769Z"
    }
```

## Create an Application Resource

Using `POST /app/`, you can create an application resource that does not have a binary associated with it. To run an assessment you must upload a binary with the same `package` name and `platform`.

```shell
    curl -H "Authorization: Bearer $NS_TOKEN" -X POST $HOST/app/ -d '{"platform": "android", "package": "com.example.app"}'
```

Like the application response, when an application resource is created it will return the apps `platform`, `package` name, associated `account`, and `created` date.

```json
    {
      "platform": "android",
      "package": "com.example.app",
      "account": "363df3a1-0e8b-4960-bb6e-cc09aced37a1",
      "created": "2016-04-08T19:06:00.257Z"
    }
```

## Get Application Details

You can also get specific application details  using `GET /app/{platform}/{package}`.


```shell
    curl -H "Authorization: Bearer $NS_TOKEN" -X GET $HOST/app/android/com.example.app
```

The response will contain the apps `platform`, `package` name, associated `account`, and `created` date.

```json
    {
      "platform": "android",
      "package": "com.example.app",
      "account": "363df3a1-0e8b-4960-bb6e-cc09aced37a1",
      "created": "2016-04-08T19:06:00.257Z"
    }
```

## View Application Configuration

Application configuration is set by `package` name. Each `package` can have its own individual application configuration.

You can view a specific applications configuration using `GET /app/{platform}/{package}/config`

```shell
    curl -H "Authorization: Bearer $NS_TOKEN" -X GET $HOST/app/android/com.example.app/config
```

The response will show which configuration options are set for that specific package.

```json
    {
      "dynamic": {
        "search_data": {
          "name": {
            "value": "Franz Kafka"
          },
          "username": {
            "value": "fkafka"
          },
          "password": {
            "value": "foobar"
          },
          "email": {
            "value": "franzkafka@metamorphosis.com"
          }
        },
        "config_options": {
          "multipass": true
        }
      },
      "static": "true"
    }
```

## Set Application Configuration

Using the `POST /app/{platform}/{package}/config` endpoint will allow the configuration for a specific application package to be set.

You can mark specific static tests `"static": { "secure_random_check": "false" }` and this command will run every test except the secure random check.

### Android Static Tests

```
    certificate_validity_check
    debug_flag_check
    keysize_check
    master_key_check
    obfuscation_check
    secure_random_check
    dynamic_code_loading_check
    application_overprivileged_check
    allow_backup_check
    javascript_interface_check
    urls_check
    decode_apk_check
    decompile_apk_check
    get_app_files
    get_native_methods
    get_reflection_code
    app_activities
    main_activity
```
### Multipass

Multipass mode is a mode which causes analysis to be done on the application multiple times to allow different "passes" of analysis to occur. For example, if we are actively man-in-the-middling an application, we may not be able to exercise the full functionality of the app if the app is cert pinning. We could run the application with all cert validations disabled, but this would not give us any insight into whether or not the app was properly checking certs. In this way, the app would need to be ran multiple times with different analysis options with and without certificate validation killed. In it's current implementation, multipass does exactly this. It runs the app once while mitming allowing whatever TLS validations are in place to take over, and once more with certificate checking disabled.

```shell
    curl -H "Authorization: Bearer $NS_TOKEN" -X POST $HOST/app/android/com.example.app/config -d '{
      "dynamic": {
        "search_data": {
          "name": {
            "value": "Franz Kafka"
          },
          "username": {
            "value": "fkafka"
          },
          "password": {
            "value": "foobar"
          },
          "email": {
            "value": "franzkafka@metamorphosis.com"
          }
        },
        "config_options": {
          "multipass": true
        }
      },
      "static": "true"
    }'
```

In this example the `name`, `username`, `email`, and `password` were set to the login credentials for the sample application. This will allow the UI Automator on the rig to successfully login to the application to further exercise the UI for test coverage.

## Trigger Application Assessment

After changing the configurations, using `POST /app/{platform}/{package}/assessment/` will trigger an assessment on the most recent build of the application with the new configuration options.

```shell
    curl -H "Authorization: Bearer $NS_TOKEN" -X POST $HOST/app/platform/com.example.app/assessment/
```

The assessment endpoint will return a response containing that specific applications `task` id.

## View Assessment Report

You can view the assessment response in json using `GET /app/{platform}/{package}/assessment/{task}/report`.

```shell
    curl -H "Authorization: Bearer $NS_TOKEN" -X GET $HOST/app/platform/com.example.app/assessment/task/report
```

The report contains all of the information from our NowSecure Automated Security Assessment. You can view the report formatted with descriptions and recommendations at https://lab.nowsecure.io/app/{platform}/{package}/assessment


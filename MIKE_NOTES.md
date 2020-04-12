# Notes

My notes on testing using schemathesis.

## Builder

1. Installed ruby. I've never really used ruby or built a rails app, so this'll be fun!
1. Ran `gem install rails` because I saw it is a rails app. This step may not be necessary, though, as there is a Gemfile, which I saw later. Then got a coffee, as it takes forever on Windows.
1. Ran `bundle install`. Saw that it needed `config/database.yml` so I killed the install made that by copying the example file. Ditto for `configuration.yml` and `additional_environment.rb`. The I relaunched `bundle install`. Got a second coffee, as this also took forever.
1. Did sanity test with `rails s`, the server starts. Noted that the default url in `config/settings.yml` is localhost:3000, which is what the server starts on. Then I pinged the root, which gave an error `(Can't connect to MySQL server on 'localhost' (10061))`.
1. Tried a dockerized version of `mysql`. Didn't work immediately, too lazy to debug so tried the `sqlite` connector.
1. This worked, but got the error `Could not find table 'settings'`. Ran `rails db:migrate` and started the server again. This worked, and now redline boots up.
1. Starting this project, it looked like it was spewing a bunch of html, so I opened my browser and pointed it to `localhost:3000`. There was a possibility to log in, and I created a username and password, but it said "waiting for admin approval". Googling around, I saw the default admin credentials were username `admin`, password `admin`. I used these and got in as admin.
1. I used the [instructions on the redmine site](https://www.redmine.org/projects/redmine/wiki/Rest_api#Authentication) to enable the REST api.
1. Smoke tested it with `curl http://admin:<password>@localhost:3000/issues.xml` and it worked.
1. I found a [swagger spec for redline](https://github.com/komikoni/redmine-swagger/blob/master/swagger.yaml).
1. Created a virtualenv and installed schemathesis. It barfed because `swagger.yaml` had some undecodable characters. I removed all the descriptions and examples.

## Runner

Run command: `schemathesis run --base-url http://admin:meeshkan@localhost:3000 .\swagger.yaml`.

```bash
schemathesis run --base-url http://admin:meeshkan@localhost:3000 .\swagger.yaml
================ Schemathesis test session starts ================
platform Windows -- Python 3.7.3, schemathesis-1.1.0, hypothesis-5.8.1, hypothesis_jsonschema-0.12.0, jsonschema-3.2.0
rootdir: C:\Users\MikeSolomon\devel\test-apis\redmine
hypothesis profile 'default' -> database=DirectoryBasedExampleDatabase('C:\\Users\\MikeSolomon\\devel\\test-apis\\redmine\\.hypothesis\\examples')
Schema location: file:///C:/Users/MikeSolomon/devel/test-apis/redmine/swagger.yaml
Base URL: http://admin:meeshkan@localhost:3000
Specification version: Swagger 2.0
Workers: 1
collected endpoints: 12

GET /issues.{format} .                                     [  8%] 
POST /issues.{format} F                                    [ 16%] 
GET /issues/{issue_id}.{format} .                          [ 25%] 
PUT /issues/{issue_id}.{format} .                          [ 33%] 
DELETE /issues/{issue_id}.{format} .                       [ 41%] 
POST /issues/{issue_id}/watchers.{format} .                [ 50%]
DELETE /issues/{issue_id}/watchers/{user_id}.{format} .    [ 58%]
GET /projects.{format} .                                   [ 66%]
POST /projects.{format} .                                  [ 75%]
GET /projects/{project_id}.{format} .                      [ 83%]
PUT /projects/{project_id}.{format} .                      [ 91%]
DELETE /projects/{project_id}.{format} .                   [100%]

============================ FAILURES ============================
_____________________ POST: /issues.{format} _____________________
1. Received a response with 5xx status code: 500

Check           : not_a_server_error
Path parameters : {'format': 'xml'}
Body            : {'issue': {'project_id': 0, 'status_id': '', 'tracker_id': 0, 'assigned_to_id': '0'}}

Run this Python code to reproduce this failure: 

    requests.post('http://admin:meeshkan@localhost:3000/issues.xml', json={'issue': {'project_id': 0, 'status_id': '', 'tracker_id': 0, 'assigned_to_id': '0'}})

Or add this option to your command line parameters: --hypothesis-seed=314496125813171442741941431919500862540
============================ SUMMARY =============================

not_a_server_error            1068 / 1096 passed          FAILED  

================= 11 passed, 1 failed in 197.34s =================
```

One command failed, so digging into it.

When I run:

```python
requests.post('http://admin:meeshkan@localhost:3000/issues.xml', json={'issue': {'project_id': 0, 'status_id': '', 'tracker_id': 0, 'assigned_to_id': '0'}})
```

There is this server error:

```
   (0.1ms)  rollback transaction
Completed 500 Internal Server Error in 808ms (ActiveRecord: 776.5ms)



NoMethodError (undefined method `assignable_users' for nil:NilClass):

app/models/issue.rb:941:in `assignable_users'
app/models/issue.rb:742:in `validate_issue'
app/controllers/issues_controller.rb:143:in `create'
lib/redmine/sudo_mode.rb:64:in `sudo_mode'
```

My ruby-foo is way to low to understand what this means, but it looks like something is `nil` that is not supposed to be. My uneducated guess is that because a project with id `0` does not exist yet, the project can't be found, but I could be wrong!

At any rate, the app probably shouldn't return a server error, so I think this'd qualify as a bug.

## Mocker

The only mock I needed to make was the database. `mysql` was too much of a pain, so I used `sqlite`. If it winds up being important, I can investigate why `mysql` was not connecting - probably some connection string issue.
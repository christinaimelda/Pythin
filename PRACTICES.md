
# Practices


## behave
- tests should verify business requirements and functionality
- create steps that do only one thing (when possible)
- chain steps together to create the desired state
- hide un-necessary data or configuration in the step itself
  e.g. don't put a persons name in the feature, unless someone reading the scenario needs to know who it was
- save data that needs to be passed from one step to another in `context`
- save the guid of people/members that were created in the `context.investigators` or `context.members` lists for easy 
  retrieval (`get_person_from_context(...)` can be used to easily retrieve the person that was created for the test)
- cleanup after successful scenarios
  - this is done automatically for the people/member(s) that were created as long as they were added into 
  context.investigators or context.members, other things that need to be cleaned up can be stored in 
  `context.change_us_after` (see `before_scenario` in `/features/environment.py` for more information)
- tags:
  - @<service_name> will instantiate the named services class in the context object when the scenario or feature is run
  - `@open_db` to open and persist the database connection for the duration of the tagged scenario or feature
  - if you use the `@wip` tag, they should all be removed before opening a PR for the branch, only working code/tests
   should be merged into the master branch :-)
- line up the spaces after the Given, When, Then and And keywords vertically in the feature files, we think it 
  just looks better that way
- IntelliJ has built in data/example table formatting, access it by right clicking on the table and clicking pipe table > format)
  - for creating complicated tables, there is a free Chrome app called [Tidy Gherkin](https://chrome.google.com/webstore/detail/tidy-gherkin/nobemmencanophcnicjhfhnjiimegjeo)
- add a story id (e.g. ABA-749, MSERV-1784) under the feature or each scenario name where the story is tested  
- keep the steps simple at first and only add variable capabilities when needed
- use a service to create/modify entities or data and only insert/change data directly in the database as little as possible
  - the services often do data/state validation or create associated data in other tables and if the database is 
    used directly incorrect/invalid data/state can be created
  - if we use the database directly we are duplicating what the service already does and increasing our maintenance load    
- use two blank lines between scenarios for enhanced readability
- write your steps somewhere between imperative and declarative styles
- don't write steps like this (for the person/member):
  `And the show on progress record flag is set for the person`
  instead write them like this:
  `And their show on progress record flag is set`
  It will make them more generic and applicable to either members or people
- the `@step('...')` step decorator can be stacked to provide matches for multiple step names, but do so carefully to 
  reduce creating confusion
  
  e.g. 
  ```
     @given('a person')
     @given('a new person')
  ```


## python
- please follow the formatting guidelines as described in [PEP 8](https://www.python.org/dev/peps/pep-0008/)
  (IntelliJ will highlight them with a squiggly underline)
- use variable type hints (not enforced at runtime) in function 
  declarations e.g. `def something_awesome(a_var: int, b_var: str, d_var: dict)`
- function comments/descriptions go _within_ the function
  - if you type triple double quotes under a function declaration and press enter IntelliJ will insert a 
    documentation section for you
- use keyword arguments when calling functions
    e.g. `get_request(url=endpoint, expected_response_code=expected_response_code, auth=self.auth)`

## Folder structure and contents
 
### `/config` folder
- `config.py` contains service and lane global configurations, change this to change which lane or database is being 
accessed by the tests
- create and maintain `env_..._service.py` files for all active services created by or used by the services team
    - update the `env_list` within a service's file as versions are added or retired
        - lists all endpoints for a service as well as all urls for the various lanes
        - for endpoints that need a parameter at the end, do a simple concatenation
        
            For example if swagger lists an endpoint like this: 
            `PUT /person-service/api/people/calculate/status/{id}`
            
            Specify it in the config like this: `/person-service/api/people/calculate/status/`
            
        - for endpoints that need a parameter in the middle of the url use curly braces and `.format(...)` to replace the 
        braces with the appropriate value
            
            For example if swagger lists and endpoint like this:
            `PUT /person-service/api/people/merge/from/{fromGuid}/to/{toGuid}`
            
            Specify it as is in the config and setup mapping of variables to the correct args:
            `... = self._build_url(self._env["mergePerson"].format(fromGuid=from_person_guid, toGuid=to_person_guid))`
    
    - functions for contacting the service using supplied payloads (no formatting of data should be done in those function)
        1. create the url
        1. send the request (with simple response code validation done within the request function)
        1. return the result

### `/feature` folder
This folder contains the [behave](https://github.com/behave/behave) features and steps
- place features for a specific service within a folder named for the service e.g. `person_service`
- contains the `environment.py` which controls how behave runs and how tags and hooks are handled

#### `.../steps` folder
This folder contains the steps used by the `*.features` in the parent directory
and should be named following this pattern: 'a service prefix' an 'optional_description_of_component' 
ending with `_steps.py`

e.g. `database-events.py` or `product_request_processor_steps.py`

Inside each step file please keep the steps organized in groups of Givens, Whens, and Thens 
It is preferred to use `@step(...)` as little as possible due to the possible confusion of what the step does. 
So @step should be replaced with the appropriate @given/@when/@then. 

Only step definitions should be in the steps folder, helper functions should be in the utils folder

__note: behave does not support sub-directories within steps__ 


### `/enum` folder
This folder is for enum classes for data that _very rarely_ changes, such as person statuses.

Please use an enum when possible instead of the value of an id so that changes can be effectively handled


### `/sandbox-personal` folder
This folder is for personal python files that you can freely change and won't be committed (the folder is ignored by 
`.gitignore`). One example would be a call to the `gherkin_formatter`


### `/sandbox-team` folder
This folder is for shared python files and examples that are not a test


### `/target` folder
This folder contains the caches of service or database responses that were created by `save_results_to_file` in 
`utils/cache_response_helper.py` (this folder is also ignored by `.gitignore` and won't exist if no cache file has been 
created on your local machine)


### `/templates` folder
This folder is for json payload templates grouped by service
 

### `/tests` folder
This folder contains tests from before behave was chosen as our test runner, as time allows these tests are being 
migrated to behave


### `/unit_tests` folder
This folder contains the behave tests for testing the custom functions located in the `/utils` folder. 


### `/utils` folder
This folder contains general and service utility functions
  - service utility functions should gather and process data and then call the applicable service function
    - e.g. a function to create a person should live here and should accept various parameters and or kwargs to create 
      that person, which is then appropriately formatted and the createNewHousehold or createNewPerson endpoint is
      called to create the person

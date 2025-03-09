# <p align="center"> Faceplate Coding Standard & Guideline

## <p align="center"> Conventions

#### Be consistent
Hold on to one style and use throughout the whole project. 

***
#### Hold on to existing code style
A consistent style throughout the entire project is better than mixing multiple styles.
Just because something looks good to you doesn't mean it will for someone else.

> [!IMPORTANT]
> It's strongly recommended to read [Erlang Efficiency Guide](https://www.erlang.org/doc/system/efficiency_guide.html) to write efficient code and avoid common mistakes.

### <p align="center"> Modules

***
#### Simplicity & Consistency 
A module should only contain what it's intended to do. Avoid mixing unrelated functionality into a single module.
Be flexible - split related functions into corresponding modules. This makes navigation and maintenance easier.
When a module tries to do too much at once, it becomes hard to maintain and navigate.

***
#### Single Responsibility Module 
The name of each module should show what it was created for and what functions it performs. Module should restrict its responsibility within abstraction it was created. 

*** 
#### Use [Facade](https://en.wikipedia.org/wiki/Facade_pattern) pattern
Implement simple API functions which will encapsulate inner module logic thus making it easier to understand which set of functions the module has.
This pattern goes well with grouping functions because it creates a clear line between external (public) and internal (private) functions.

### <p align="center"> Functions

***
#### [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle
Avoid duplicating code in your module or throughout one application.  

***
#### [KISS](https://en.wikipedia.org/wiki/KISS_principle) principle
Make it simple, don't overengineer solutions thus making it harder to understand your code. 

***
#### Naming
Use `snake_case` for naming functions using characters and numbers.
```erlang
start() -> ...
start_link() -> ...

read() -> ...
bulk_read() -> ...

check_something() -> ...
loop_something() -> ...
do_something() -> ...
try_something() -> ...

string_to_number() -> ...
string2number() -> ...
```

***
#### Grouping
Separate your functions which are exported and which are not. 
Exported functions act like an API for a module and should be places first. 
All other functions will go after exported functions and will be grouped as internal / helper functions.
```erlang
-export([
  my_api_fun/1
]).

%%% External (public) functions
my_api_fun(Arg) ->
  my_internal_fun(Arg),
  ok.

%%% Internal (private) functions
my_internal_fun(Arg) ->
  ok.
```

***
#### Too much arguments
When there are too much arguments in your function, records can be used.
It allows us to ignore the order of arguments and thus making it harder to accidentally mix up arguments.
```erlang
% BAD
fun(Owner, List, Options, Data) -> ...

% GOOD
-record(fun_data, {
  owner,
  list,
  options,
  data
}).
fun(#fun_data{} = FunData) -> ...
```

***
#### Return value
The function return value should be explicitly shown. Return value should represent if the function operated successfully or not.
Otherwise, it can be hard to understand what is expected from a function. 
```erlang
%% BAD
fun() ->
  % Processing...
  some_random_fun().

%% OK: You can use pattern matching to explicitly expect some kind of successful format.  
fun() ->
  {ok, Result} = some_random_fun(),
  Result.

%% GOOD
fun() ->
  case some_random_fun() of
    {ok, Result} -> Result;
    {error, Error} -> throw({error, failed_to_run_fun})
  end.

fun() ->
  try
    % Your pipeline of functions...
    some_random_fun(),
    ok
  catch
    _Exception:Error ->
      ?LOGERROR(...),
      {error, failed_to_run_fun}
  end.
```

### <p align="center"> Variables

***
#### Naming
`CamelCase` must be used for naming. Don't use any other special characters, numbers and etc.
Be simple and meaningful at the same time.

```erlang
bad(A) ->
  A1 = A,
  A2 = A1.

good(Input) ->
  Output = Input + 1.
good(SettingsIn) ->
  Settings = check_settings(SettingsIn),
  SettingsOut = merge_default_settings(Settings),
  SettingsOut.
good(ArchivePath) ->
  ArchiveOID = ?OID(ArchivePath),
  ArchiveOID.
```
It is also good in some cases to add postfix to a variable name to indicate which data type is expected from it. 
```erlang
fun(InputStr) when is_binary(InputStr) ->
  InputInt = erlang:binary_to_integer(InputStr),
  InputInt.
```
Avoid creating situations like this one (oversimplified example): 
```erlang
fun(Integer) ->
  List = Integer,
  other_fun(List),
  ok.
```

***
#### Don't use ignored variables
Never use variables starting with `'_'` anywhere. They are intentionally left that way to indicate the variable isn't used after.
The naming after `'_'` are used to clearly indicate what that argument is. 

*** 
#### Prefer binaries 
When binaries exceed size of 64-bytes, it is not copied, but pointer is used when passed as an argument to functions or to a local process.   

***
#### Lowercase atoms
Use `snake_case` to all atoms. Try to maximize the usage of one atom name without losing meaning, but don't be afraid to create a new one when needed. 

***
#### Dynamically generated atoms
While it's okay to create atoms during compilation, creating atoms during runtime and generating them should be avoided. 
> [!CAUTION]
> You should beware that **atoms aren't garbage-collected** and live until the Erlang VM is terminated.
> It means that if you create atoms dynamically, you will use lots of memory.
> Using too much memory can lead to killing your process or the whole Erlang VM.

### <p align="center"> Macros
> [!NOTE]
> Macros should be declared after function exports and before function implementations, just alongside with records.

***
#### Naming
'ALL_MUST_BE_UPPERCASE' convention is used to name macros. 

***
#### Don't overdo 
Overusing and deepening the level of the macro makes it hard to debug and identify the problem. 
Optimal depth of the macro should be one with specific expections when deepening macros is needed.
```erlang
% BAD
-define(MACRO_C, MACRO_D).
-define(MACRO_B, MACRO_C).
-define(MACRO_A, MACRO_B).

% OK
-define(BASE_PATH, <<"/root/.patterns">>).
-define(ARCHIVE, <<(?BASE_PATH)/binary, "/archive">>).
```

### <p align="center"> Conditions

***
#### Use case instead of if
`case` is more convenient and easier to read than `if` because of its syntax and pattern matching.

#### Don't spaghettify
You can easily overuse cases and create hard-to-read code blocks by nesting cases which can lead to bugs and confusion.

### <p align="center"> Records

***
#### Naming
Use snake_case for naming records. Separate words with '_'.
```erlang
-record(state, {
  name,
  archive_oid,
  current_status
}).
```

***
#### Don't use them inside macros
```erlang
% BAD
-define(STATE, #state{}).
```

***
#### Declare before functions 
Records and macros must be declared **before** function implementions.

***
#### Don't share records
Records intended to be used inside the module and only. Sending records as argument to other module functions or processes is a bad idea.

### <p align="center"> Commenting

> [!IMPORTANT]
> Your code itself should be self-explanotary in the first place. No need to explain your steps through massive comments.

*** 
#### Don't overdo
Comments should answer to questions why and explain edge-cases, but not explaining everything with how and what. 
Too much comments literally litters the code. 

*** 
#### Leveling comments
Try to level and organize your comments through leveling them corresponding to:
```erlang
%%% Module level
%% Function level
% Function body level
```

### <p align="center"> Error Handling

***
#### Returning reasonable errors
Write meaningful ```error``` messages that clearly indicate where and why an error happened.
```erlang
%% Reasoning: We blindly accept the error and throw it away without any context.
bad() ->
  case fun() of
    {ok, GoodResult} ->
      ok;
    Error ->
      throw(Reason)
  end.
%% Reasoning: The error is returned, but without explanation.
bad() ->
  case fun() of
    {ok, GoodResult} ->
      ok;
    Error ->
      {error, Error}
  end.
%% Reasoning: This tells the programmer nothing. The error can be mistaken
%% for a native OTP error.
bad() ->
  badarg.

%% Function checks the argument type and handles errors clearly.
good(List) when is_list(List) ->
  %% Handling list...
  ok;
good(List) ->
  throw({invalid_arg, {list, List}}).
%% Or this one
good(List) ->
  {error, {invalid_arg, {list, List}}}.
```

*** 
#### Let errors be visible
Log every error to the log providing information to identify the problem.
Prefer using macros: ```?LOGINFO```, ```?LOGWARNING```, ```?LOGERROR``` 

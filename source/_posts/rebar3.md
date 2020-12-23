---
title: rebar3
date: 2020-12-23 10:56:10
tags: rebar3
categories:
- rebar3
- erlang
---


## rebar3 项目配置

* Makefile
* tool.sh
* rebar.config
* rebar.config.script
* sys.config.src
* vm.args.src
* <APP_NAME>.app.src.script

### Required

* rebar 3.14.3 on Erlang/OTP 22 Erts 10.7.2.1

### Makefile文件

```shell
all: compile dialyzer test

###===================================================================
### build
###===================================================================
.PHONY: co compile es escriptize run

co:compile
compile:
	rebar3 compile

es:escriptize
escriptize: clean
	rebar3 escriptize

### clean
.PHONY: clean distclean
clean:
	rebar3 clean

distclean:
	rebar3 clean -a

###===================================================================
### test
###===================================================================
.PHONY: test eunit ct testclean

test: epmd dialyzer
	rebar3 do eunit, ct --config test/ct/ct.config --sys_config config/test.config, cover

eunit: epmd
	rebar3 do eunit -v, cover

ct: epmd
	rebar3 do ct -v --config test/ct/ct.config --sys_config config/test.config, cover

testclean:
	@rm -fr _build/test

shell: epmd config
	rebar3 as test shell

config: epmd
	tool.sh replace_config

dialyzer: epmd
	rebar3 do dialyzer

tar: epmd
	rm -rf _build/prod
	rebar3 as prod release
	rebar3 as prod tar
###===================================================================
### other
###===================================================================
.PHONY: epmd

epmd:
	@pgrep epmd 2> /dev/null > /dev/null || epmd -daemon || true
```

### tool.sh

```shel
#!/bin/sh
# Replace environment variables
ONE="$1"
TWO="$2"
THREE="$3"
set_var(){
    export APP_LOG_ROOT="logs"
	export APP_CONSOLE_LOG_LEVEL="error"
    export APP_PORT="8081"
    export APP_NODE_NAME="push@127.0.0.1"
    export APP_COOKIE="xxx"

}
# sh tool.sh relup OLD_RELEASE NEW_RELEASE
relup()
{
	rm -rf _build/prod
	git checkout ${TWO}
	rebar3 as prod release
	git checkout ${THREE}
	rebar3 as prod release
	rebar3 as prod appup generate
	rebar3 as prod relup tar
}
replace_os_vars() {
    awk '{
        while(match($0,"[$]{[^}]*}")) {
            var=substr($0,RSTART+2,RLENGTH -3)
            slen=split(var,arr,":-")
            v=arr[1]
            e=ENVIRON[v]
            if(slen > 1 && e=="") {
                i=index(var, ":-"arr[2])
                def=substr(var,i+2)
                gsub("[$]{"var"}",def)
            } else {
                gsub("[$]{"var"}",e)
            }
        }
    }1' < "$1" > "$2"
}
replace_config(){
    set_var
    replace_os_vars config/sys.config.src config/sys.config
    replace_os_vars config/vm.args.src config/vm.args
}

case $1 in
replace_config) replace_config;;
*)
echo "Usage: sh tool.sh [command]"
echo "command:"
echo "replace_config       override sys.config and vm.args"
echo "relup release-0.1.0 release-0.1.1               genarate appup tar"
;;
esac%
```

### rebar.config

```erlang
{erl_opts, [
  {parse_transform, lager_transform},
  warn_export_all,
  warn_export_vars,
  warn_obsolete_guard,
  warn_shadow_vars,
  warn_unused_function,
  warn_deprecated_function,
  warn_unused_import,
  warnings_as_errors
]}.
{minimum_otp_vsn, "20"}.
{deps, [
  recon,
  lager,
  observer_cli
  ]}.

{eunit_opts, [export_all]}.% same as options for eunit:test(Tests, ...)
{eunit_tests, []}. % same as Tests argument in eunit:test(Tests, ...)
{cover_enabled, true}.
{dist_node, [
    {name, 'app@127.0.0.1'},
    {setcookie, 'xxxx'}
]}.
{relx, [{release, {<APP_NAME>, "read from VERSION"},
         [<APP_NAME>,
          sasl]},
        %% automatically picked up if the files
        %% exist but can be set manually, which
        %% is required if the names aren't exactly
        %% sys.config and vm.args
        {sys_config_src, "./config/sys.config.src"},
        {vm_args_src, "./config/vm.args.src"},
        %% the .src form of the configuration files do
        %% not require setting RELX_REPLACE_OS_VARS
        %% {sys_config_src, "./config/sys.config.src"},
        %% {vm_args_src, "./config/vm.args.src"}
        {overlay, [
            {mkdir, "bin/extensions"},
            {mkdir, "tmp"},
            {copy, "tool.sh", "tool.sh"},
            {copy, "hooks/pre_start.sh", "bin/hooks/pre_start.sh"},
            {copy, "hooks/pre_stop.sh", "bin/hooks/pre_stop.sh"},
            {copy, "hooks/pre_install_upgrade.sh", "bin/hooks/pre_install_upgrade.sh"},
            {copy, "hooks/post_start.sh", "bin/hooks/post_start.sh"},
            {copy, "hooks/post_stop.sh", "bin/hooks/post_stop.sh"},
            {copy, "hooks/post_install_upgrade.sh", "bin/hooks/post_install_upgrade.sh"},
            {copy, "scripts/extensions/status", "bin/extensions/status"}]},
        {extended_start_script, true},
        {extended_start_script_hooks, [
            {pre_start,
                [{custom, "hooks/pre_start.sh"}]},
            {post_start, [
                wait_for_vm_start,
                {custom, "hooks/post_start.sh"},
                {pid, "tmp/<APP_NAME>.pid"}
            ]},
            {pre_stop, [
                {custom, "hooks/pre_stop.sh"}
            ]},
            {post_stop, [
                {custom, "hooks/post_stop.sh"}
            ]},
            {pre_install_upgrade,
                [{custom, "hooks/pre_install_upgrade.sh"}]},
            {post_install_upgrade,
                [{custom, "hooks/post_install_upgrade.sh"}]}
        ]}
        ,
        {extended_start_script_extensions, [
            {do, "extensions/status"}
        ]}
        ]}.

{profiles, [
 {test, [{erl_opts, [{d, 'TEST'},nowarn_export_all, export_all]},
          {shell, [{config, "config/sys.config"}]},
          {deps, [recon, meck]}]},
  {prod, [{relx,
                     [%% prod is the default mode when prod
                      %% profile is used, so does not have
                      %% to be explicitly included like this
                      % {mode, prod},
                      {include_src, false},
                      {include_erts, "override by rebar.config.script"},
                      {system_libs, "override by rebar.config.script"}
                      %% use minimal mode to exclude ERTS
                      %  {mode, minimal}
                     ]
            },{
              deps,[
                %% already compile with centos
              {jiffy,{git, "https://git.xxxx/library/jiffy.git", {tag, "release-0.1.0"}}}
              ]
            }
          ]}]}.

{provider_hooks, [
  {pre, [{tar, {appup, tar},
  {compile, {pc, compile},
  {clean, {pc, clean}}}}]},

  {post, [{compile, {appup, compile}},
  {clean, {appup, clean}}]}
]}.
{plugins, [pc, rebar3_appup_plugin]}.
```

### rebar.config.script

```shell
%% Update relx config
Relx0 = lists:keyfind(relx, 1, CONFIG),
{relx, [{release, {<APP_NAME>, _}, RelxApps0} | RelxT0]} = Relx0,
{ok, VersionBin} = file:read_file(<<"VERSION">>),
Version = string:strip(string:strip(binary_to_list(VersionBin),both,$\n)),
Relx = {relx, [{release, {<APP_NAME>, Version}, RelxApps0}] ++ RelxT0},
NewConfig=lists:keyreplace(relx, 1, CONFIG, Relx),
%% profile
{_,Profiles} = lists:keyfind(profiles, 1, NewConfig),
Prod= proplists:get_value(prod, Profiles),
ProdRelx = proplists:get_value(relx, Prod),
ProdRelx1 = lists:keyreplace(include_erts, 1, ProdRelx, {include_erts, os:getenv("ERTS_PATH")}),
ProdRelx2 = lists:keyreplace(system_libs, 1, ProdRelx1, {system_libs, os:getenv("ERTS_PATH") ++ "/lib"}),
Prod1 = lists:keyreplace(relx, 1, Prod, {relx, ProdRelx2}),
Profiles1 = lists:keyreplace(prod, 1, Profiles, {prod, Prod1}),
NewConfig1 = lists:keyreplace(profiles, 1, NewConfig, {profiles, Profiles1}),
file:write_file("rebar.config.gen", io_lib:format("%%% auto generate ~n~p", [NewConfig1])),
NewConfig1.
```

### sys.config.src

```erlang
[
 {lager, [
    {log_root, "${APP_LOG_ROOT}"},
    %% Default handlers for lager/lager_event
    {colored, true},

    {async_threshold, 5000},
    {async_threshold_window, 500},
    {error_logger_flush_queue, true},
    {error_logger_flush_threshold, 1000},
    {error_logger_hwm, 200},
    {killer_hwm, 10000},
    {killer_reinstall_after, 5000},

    {crash_log, "crash.log"},
    {crash_log_msg_size, 65536},
    {crash_log_size, 524288000},
    {crash_log_date, "$D0"},
    {crash_log_count, 30},

    {handlers, [
        {lager_console_backend, [{level, ${APP_CONSOLE_LOG_LEVEL}}, {formatter, lager_default_formatter}, {formatter_config, [{eol, "\e[0m\r\n"}]}]},
        {lager_file_backend, [{level, "=error"}, {file, "error.log"}, {size, 524288000}, {date, "$D0"}, {count, 30}]},
        {lager_file_backend, [{level, "=warning"}, {file, "warning.log"}, {size, 524288000}, {date, "$D0"}, {count, 30}]},
        {lager_file_backend, [{level, "=info"}, {file, "info.log"}, {size, 524288000}, {date, "$D0"}, {count, 30}]}
        %%{lager_file_backend, [{level, "=debug"}, {file, "debug.log"}, {size, 10485760}, {date, "$D0"}, {count, 30}]}
    ]}
]
},
    {sync, [
    {growl, all},
    {log, all},
    {non_descendants, fix},
    {executable, auto},
    {whitelisted_modules, []},
    {excluded_modules, []}
  ]},
  {<APP_NAME>, [
      {port, ${APP_PORT}}
      ]}
 ].%
```

### vm.args.src

```erlang
-name ${APP_PUSH_NODE_NAME}
-setcookie ${APP_PUSH_COOKIE}
%% 是否启用的kernel的poll机制，默认为false
+K true
%% 异步线程池的线程数，范围为0~1024，默认为10
+A30
%% 最大进程数，范围为1024-134217727 ，默认为  262144
+P 1000000
%% 同时打开的端口数量限制（Open ports）
+Q 100000
```

### <APP_NAME>.app.src.script

```erlang
[{application, Name, L}] = CONFIG,
{ok, VersionBin} = file:read_file(<<"VERSION">>),
Version = string:strip(string:strip(binary_to_list(VersionBin),both,$\n)),
NewConfig = {application, Name, lists:keystore(vsn,1,L,{vsn, Version})},
file:write_file("apps/<APP_NAME>/src/<APP_NAME>.app.src.gen", io_lib:format("%%% auto generate ~n~p", [NewConfig])),
NewConfig.
```
---
title: ra
date: 2021-04-13 10:45:34
categories:
- erlang
tags: raft
---
    ra是由rabbitmq团队正在开发的轻量级分布式erlang集群，旨在解决分布式一致性问题，分布式集群一般分为中心华的和去中心华的，
中心化使用的算法有Paxos、raft(Paxos算法的变种),raft算法更易于理解和实现。用raft实现的服务发现的程序consul、ectd等都是raft实现的。
本文主要介绍raft在erlang中的实现，主要分析ra源码，对raft算法的具体概念请自行查阅相关文献。

## 工具和环境

- rebar 3.14.4 on Erlang/OTP 22 Erts 10.7.2.1
- [ra简单kv-store使用例子](https://github.com/rabbitmq/ra-kv-store.git)
- [ra-1.1.9](https://github.com/rabbitmq/ra.git)
- [调试跟踪工具 recon-2.5.1](https://github.com/ferd/recon)

## 分析回调

- 回调注册： 
```erlang
    Servers =  [{ra_kv1, 'ra_kv_store@127.0.0.1'}],
    ClusterId = <<"ra_kv_store">>,
    Config = #{},
    %% ra_kv_store是回调模块，实现了ra_machine behavior
    Machine = {module, ra_kv_store, Config},
   {ok, Started, Failed} = ra:start_cluster(default, ClusterId, Machine, Servers),
```
- process_command回调方法:
```erlang

   init(_Config) ->
       #{}.
    %% 执行 ra:process_command(ra_kv1, {write, Key, Value})  
    %% 会回调该方法
   apply(#{index := Index, term := Term} = _Metadata, {write, Key, Value}, State) ->
       NewState = maps:put(Key, Value, State),
       SideEffects = side_effects(Index, NewState),
       %% return the index and term here as a result
       {NewState, {Index, Term}, SideEffects};
       %% 执行 ra:process_command(ra_kv1, {cas, Key, ExpectedValue, NewValue})
       %% 会回调该方法
   apply(#{index := Index, term := Term} = _Metadata,
         {cas, Key, ExpectedValue, NewValue},
         State) ->
       {NewState, ReadValue} =
           case maps:get(Key, State, undefined) of
               ExpectedValue ->
                   {maps:put(Key, NewValue, State), ExpectedValue};
               ValueInStore ->
                   {State, ValueInStore}
           end,
       SideEffects = side_effects(Index, NewState),
       {NewState, {{read, ReadValue}, {index, Index}, {term, Term}}, SideEffects}.
  ```

## 客户端命令执行流程

- leader（ra_server_proc进程）
- 写本地wal（ra_log_wal进程）
- log replicated(ra_server_proc进程发送append_entries_rpc到别的follower)
- commited(ra_server_proc进程收到大部分follower返回append_entries_reply成功)
- machine apply(ra_server_proc进程执行ra_kv_store:apply回调)

### leader进程

  进程模块在ra_server_proc, 跟踪调用的函数栈:
  ```erlang
  18:4:15.090365 <0.2624.0> ra_kv_store:write(ra_kv1, <<"a">>, <<"2">>)

18:4:15.090878 <0.2624.0> ra:process_command(ra_kv1, {write,<<"a">>,<<"2">>})

18:4:15.091004 <0.2624.0> ra:process_command(ra_kv1, {write,<<"a">>,<<"2">>}, 5000)

18:4:15.091131 <0.2624.0> ra_server_proc:command(ra_kv1, {'$usr',{write,<<"a">>,<<"2">>},await_consensus}, 5000)

18:4:15.091429 <0.2624.0> ra_server_proc:leader_call(ra_kv1, {command,normal,{'$usr',{write,<<"a">>,<<"2">>},await_consensus}}, 5000)

18:4:15.102065 <0.2624.0> ra_server_proc:gen_statem_safe_call(ra_kv1, {leader_call,{command,normal,
                      {'$usr',{write,<<"a">>,<<"2">>},await_consensus}}}, 5000)

18:4:15.102492 <0.554.0> ra_server_proc:leader({call,{<0.2625.0>,#Ref<0.4247252258.4035444737.61292>}}, {leader_call,{command,normal,
                      {'$usr',{write,<<"a">>,<<"2">>},await_consensus}}}, {state,
..............

  ```

  代码跟踪到下面的函数(ra_server_proc模块338行):

  ```erlang
  leader(EventType, {command, normal, {CmdType, Data, ReplyMode}},
       #state{server_state = ServerState0} = State0) ->
    %% normal priority commands are written immediately
    Cmd = make_command(CmdType, EventType, Data, ReplyMode),
    %% Effects = [{send_rpc,
    %% {ra_kv2,'ra_kv_store2@127.0.0.1'},
    %%     {append_entries_rpc,1,
    %%         {ra_kv1,'ra_kv_store1@127.0.0.1'},
    %%         6,6,1,
    %%         [{7,1,
    %%           {'$usr',
    %%               #{from => {<0.2625.0>,#Ref<0.4247252258.4035444737.61292>},
    %%                 ts => 1618308255090},
    %%               {write,<<"a">>,<<"2">>},
    %%               await_consensus}}]}}]
    {leader, ServerState, Effects} =
    %% 主要调用ra_log_wal:write/5写本地wal
        ra_server:handle_leader({command, Cmd}, ServerState0),
    {State, Actions} =
    %% 该函数里面执行log replicate，会给follower发送append_entries_rpc
        ?HANDLE_EFFECTS(Effects, EventType,
                        State0#state{server_state = ServerState}),
    {keep_state, State, Actions};
  ```

  ### ra_log_wal进程

   主要处理log的写入, entry会写入到ets表且flush到*.wal文件中

跟踪函数栈:
```erlang
18:4:15.110420 <0.554.0> ra_log_wal:write({<<"RA_KV_4K9N2O5QOHEB">>,<0.554.0>}, ra_log_wal, 7, 1, {'$usr',#{from => {<0.2625.0>,#Ref<0.4247252258.4035444737.61292>},
          ts => 1618308255090},
        {write,<<"a">>,<<"2">>},
        await_consensus})

18:4:15.110735 <0.554.0> ra_log_wal:named_cast(ra_log_wal, {append,{<<"RA_KV_4K9N2O5QOHEB">>,<0.554.0>},
        7,1,
        {'$usr',#{from => {<0.2625.0>,#Ref<0.4247252258.4035444737.61292>},
                  ts => 1618308255090},
                {write,<<"a">>,<<"2">>},
                await_consensus}})

18:4:15.111202 <0.554.0> ra_log_wal:named_cast(<0.542.0>, {append,{<<"RA_KV_4K9N2O5QOHEB">>,<0.554.0>},
        7,1,
        {'$usr',#{from => {<0.2625.0>,#Ref<0.4247252258.4035444737.61292>},
                  ts => 1618308255090},
                {write,<<"a">>,<<"2">>},
                await_consensus}})

18:4:15.132713 <0.542.0> ra_log_wal:handle_batch([{cast,{append,{<<"RA_KV_4K9N2O5QOHEB">>,<0.554.0>},
               7,1,
........
```

具体进程内部实现关键代码:

```erlang

-spec handle_batch([wal_op()], state()) ->
    {ok, [gen_batch_server:action()], state()}.
handle_batch(Ops, State0) ->
    State = lists:foldr(fun handle_op/2, start_batch(State0), Ops),
    %% process all ops
    {ok, [garbage_collect], complete_batch(State)}.

.....

handle_msg({append, {UId, Pid} = Id, Idx, Term, Entry},
           #state{writers = Writers} = State0) ->
    case maps:find(UId, Writers) of
        {ok, {_, PrevIdx}} when Idx =< PrevIdx + 1 ->
            write_data(Id, Idx, Term, Entry, false, State0);
        error ->
            write_data(Id, Idx, Term, Entry, false, State0);
        {ok, {out_of_seq, _}} ->
            % writer is out of seq simply ignore drop the write
            % TODO: capture metric for dropped writes
            State0;
        {ok, {in_seq, PrevIdx}} ->
            % writer was in seq but has sent an out of seq entry
            % notify writer
            ?DEBUG("WAL: requesting resend from `~w`, "
                   "last idx ~b idx received ~b",
                   [UId, PrevIdx, Idx]),
            Pid ! {ra_log_event, {resend_write, PrevIdx + 1}},
            State0#state{writers = Writers#{UId => {out_of_seq, PrevIdx}}}
    end;
.......
%% 组织每一个entry的数据结构
write_data({UId, _} = Id, Idx, Term, Data0, Trunc,
           #state{conf = #conf{compute_checksums = ComputeChecksum},
                  wal = #wal{writer_name_cache = Cache0,
                             entry_count = Count} = Wal} = State00) ->
    EntryData = to_binary(Data0),
    EntryDataLen = byte_size(EntryData),
    %% entry的头字段
    {HeaderData, HeaderLen, Cache} = serialize_header(UId, Trunc, Cache0),
    % fixed overhead =
    % 24 bytes 2 * 64bit ints (idx, term) + 2 * 32 bit ints (checksum, datalen)
    DataSize = HeaderLen + 24 + EntryDataLen,
    % if the next write is going to exceed the configured max wal size
    % we roll over to a new wal.
    case should_roll_wal(DataSize, State00) of
        true ->
            %% 如果wal文件已经达到最大值
            %% 马上处理完该wal文件并且产生一个新的序号wal文件
            %% 把文件的Ref指向新的文件
            State = roll_over(State00),
            % TODO: there is some redundant computation performed by
            % recursing here it probably doesn't matter as it only happens
            % when a wal file fills up
            write_data(Id, Idx, Term, Data0, Trunc, State);
        false ->
            State0 = State00#state{wal = Wal#wal{writer_name_cache = Cache,
                                                 entry_count = Count + 1}},
            Entry = [<<Idx:64/unsigned,
                       Term:64/unsigned>>,
                     EntryData],
            Checksum = case ComputeChecksum of
                           true -> erlang:adler32(Entry);
                           false -> 0
                       end,
            Record = [HeaderData,
                      <<Checksum:32/integer, EntryDataLen:32/unsigned>>,
                      Entry],
            append_data(State0, Id, Idx, Term, Data0,
                        DataSize, Record, Trunc)
    end.

.......
%% 组织数据和ets表不存在创建ets表
append_data(#state{conf = Cfg,
                   file_size = FileSize,
                   batch = Batch0,
                   writers = Writers} = State,
            {UId, Pid}, Idx, Term, Entry, DataSize, Data, Truncate) ->
    Batch = incr_batch(Cfg, Batch0, UId, Pid,
                       {Idx, Term}, Data, Entry, Truncate),
    State#state{file_size = FileSize + DataSize,
                batch = Batch,
                writers = Writers#{UId => {in_seq, Idx}} }.

......



complete_batch(#state{batch = undefined} = State) ->
    State;
complete_batch(#state{batch = #batch{waiting = Waiting,
                                     writes = NumWrites},
                      conf = #conf{open_mem_tbls_name = OpnTbl} = Cfg
                      } = State00) ->
    % TS = os:system_time(microsecond),
    %% flush到*.wal文件
    State0 = flush_pending(State00),
    % SyncTS = os:system_time(microsecond),
    counters:add(Cfg#conf.counter, ?C_WRITES, NumWrites),
    State = State0#state{batch = undefined},

    %% process writers
    _ = maps:map(fun (Pid, #batch_writer{tbl_start = TblStart,
                                         uid = UId,
                                         from = From,
                                         to = To,
                                         term = Term,
                                         inserts = Inserts,
                                         tid = Tid}) ->
                         %% need to reverse inserts in case an index overwrite
                         %% came to be processed in the same batch.
                         %% Unlikely, but possible
                         %% 写一份entry到ets表内
                         true = ets:insert(Tid, lists:reverse(Inserts)),
                         true = ets:update_element(OpnTbl, UId,
                                                   [{2, TblStart}, {3, To}]),
                        %% 给ra_server_proc进程发送消息
                         Pid ! {ra_log_event, {written, {From, To, Term}}},
                         ok
                 end, Waiting),
    State.

%% 写入到*.wal文件
flush_pending(#state{wal = #wal{fd = Fd},
                     batch = #batch{pending = Pend} = Batch,
                     conf =  #conf{write_strategy = WriteStrategy,
                                   sync_method = SyncMeth}} = State0) ->

    case WriteStrategy of
        default ->
            ok = ra_file_handle:write(Fd, Pend),
            ok = ra_file_handle:SyncMeth(Fd),
            ok;
        o_sync ->
            ok = ra_file_handle:write(Fd, Pend)
    end,
    State0#state{batch = Batch#batch{pending = []}}.


```

### log replicate

执行append_entries_rpc入口函数(ra_server_proc模块345行)跟踪:
```erlang
18:4:15.126833 <0.554.0> ra_server_proc:handle_effects(leader, [{send_rpc,
     {ra_kv2,'ra_kv_store2@127.0.0.1'},
     {append_entries_rpc,1,
         {ra_kv1,'ra_kv_store1@127.0.0.1'},
         6,6,1,
         [{7,1,
           {'$usr',
               #{from => {<0.2625.0>,#Ref<0.4247252258.4035444737.61292>},
                 ts => 1618308255090},
               {write,<<"a">>,<<"2">>},
               await_consensus}}]}}], {call,{<0.2625.0>,#Ref<0.4247252258.4035444737.61292>}}, {state,
               ......
8:4:15.132193 <0.554.0> ra_server_proc:send({ra_kv2,'ra_kv_store2@127.0.0.1'}, {'$gen_cast',
    {append_entries_rpc,1,
        {ra_kv1,'ra_kv_store1@127.0.0.1'},
        6,6,1,
        [{7,1,
          {'$usr',
              #{from => {<0.2625.0>,#Ref<0.4247252258.4035444737.61292>},
                ts => 1618308255090},
              {write,<<"a">>,<<"2">>},
              await_consensus}}]}}, {conf,"{ra_kv1,'ra_kv_store1@127.0.0.1'}",ra_kv1,<<"ra_kv_store">>,100,1000,
      30000,undefined,25,1000000,30000,1000,
      {atomics,#Ref<0.4247252258.4035575810.46285>}})

18:4:15.132627 <0.554.0> ra_server_proc:send/3 --> ok
......
```

关键代码片段:

```erlang
handle_effects(RaftState, Effects0, EvtType, State0) ->
    handle_effects(RaftState, Effects0, EvtType, State0, []).
% effect handler: either executes an effect or builds up a list of
% gen_statem 'Actions' to be returned.
handle_effects(RaftState, Effects0, EvtType, State0, Actions0) ->
    {State, Actions} = lists:foldl(
                         fun(Effects, {State, Actions}) when is_list(Effects) ->
                                 handle_effects(RaftState, Effects, EvtType,
                                                State, Actions);
                            (Effect, {State, Actions}) ->

                                 handle_effect(RaftState, Effect, EvtType,
                                               State, Actions)
                         end, {State0, Actions0}, Effects0),
    {State, lists:reverse(Actions)}.

handle_effect(_, {send_rpc, To, Rpc}, _,
              #state{conf = Conf} = State0, Actions) ->
    % fully qualified use only so that we can mock it for testing
    % TODO: review / refactor to remove the mod call here
    case ?MODULE:send_rpc(To, Rpc, State0) of
        ok ->
            {State0, Actions};
        nosuspend ->
            %% update peer status to suspended and spawn a process
            %% to send the rpc without nosuspend so that it will block until
            %% the data can get through
            Self = self(),
            _Pid = spawn(fun () ->
                                 %% AFAIK none of the below code will throw and
                                 %% exception so we should always end up setting
                                 %% the peer status back to normal
                                 ok = gen_statem:cast(To, Rpc),
                                 incr_counter(Conf, ?C_RA_SRV_MSGS_SENT, 1),
                                 Self ! {update_peer, To, #{status => normal}}
                         end),
            {update_peer(To, #{status => suspended}, State0), Actions};
        noconnect ->
            %% for noconnects just allow it to pipeline and catch up later
            {State0, Actions}
    end;

```

接受append_entries_reply函数入口:
```erlang
18:4:15.150726 <0.554.0> ra_server_proc:leader(cast, {{ra_kv2,'ra_kv_store2@127.0.0.1'},{append_entries_reply,1,true,8,7,1}}, {state,
    {conf,"{ra_kv1,'ra_kv_store1@127.0.0.1'}",ra_kv1,<<"ra_kv_store">>,100,
    .......


```
关键代码片段ra_server_proc模块444行:

```erlang

leader(EventType, Msg, State0) ->
    case handle_leader(Msg, State0) of
        {leader, State1, Effects1} ->
            {State, Actions} = ?HANDLE_EFFECTS(Effects1, EventType, State1),
            {keep_state, State, Actions};
        {follower, State1, Effects1} ->
            {State, Actions} = ?HANDLE_EFFECTS(Effects1, EventType, State1),
            Monitors = ra_monitors:remove_all(machine, State#state.monitors),
            next_state(follower, State#state{monitors = Monitors}, Actions);
        {stop, State1, Effects} ->
            % interact before shutting down in case followers need
            % to know about the new commit index
            {State, _Actions} = ?HANDLE_EFFECTS(Effects, EventType, State1),
            {stop, normal, State};
        {delete_and_terminate, State1, Effects} ->
            {State2, Actions} = ?HANDLE_EFFECTS(Effects, EventType, State1),
            State = send_rpcs(State2),
            case ra_server:is_fully_replicated(State#state.server_state) of
                true ->
                    {stop, {shutdown, delete}, State};
                false ->
                    next_state(terminating_leader, State, Actions)
            end;
        {await_condition, State1, Effects1} ->
            {State, Actions} = ?HANDLE_EFFECTS(Effects1, EventType, State1),
            next_state(await_condition, State, Actions)
    end.

%% ra_server模块341行:
    handle_leader({PeerId, #append_entries_reply{term = Term, success = true,
                                             next_index = NextIdx,
                                             last_index = LastIdx}},
              #{current_term := Term,
                cfg := #cfg{id = Id,
                            log_id = LogId} = Cfg} = State0) ->
    ok = incr_counter(Cfg, ?C_RA_SRV_AER_REPLIES_SUCCESS, 1),
    case peer(PeerId, State0) of
        undefined ->
            ?WARN("~s: saw append_entries_reply from unknown peer ~w",
                  [LogId, PeerId]),
            {leader, State0, []};
        Peer0 = #{match_index := MI, next_index := NI} ->
            Peer = Peer0#{match_index => max(MI, LastIdx),
                          next_index => max(NI, NextIdx)},
            State1 = put_peer(PeerId, Peer, State0),
            {State2, Effects0} = evaluate_quorum(State1, []),

            {State3, Effects1} = process_pending_consistent_queries(State2,
                                                                    Effects0),

            {State, More, RpcEffects0} = make_pipelined_rpc_effects(State3, []),
            % rpcs need to be issued _AFTER_ machine effects or there is
            % a chance that effects will never be issued if the leader crashes
            % after sending rpcs but before actioning the machine effects
            RpcEffects = case More of
                             true ->
                                 [{next_event, info, pipeline_rpcs} |
                                  RpcEffects0];
                             false ->
                                 RpcEffects0
                         end,
            Effects = Effects1 ++ RpcEffects,
            case State of
                #{cluster := #{Id := _}} ->
                    % leader is in the cluster
                    {leader, State, Effects};
                #{commit_index := CI,
                  cluster_index_term := {CITIndex, _}}
                  when CI >= CITIndex ->
                    % leader is not in the cluster and the new cluster
                    % config has been committed
                    % time to say goodbye
                    ?INFO("~s: leader not in new cluster - goodbye", [LogId]),
                    {stop, State, Effects};
                _ ->
                    {leader, State, Effects}
            end
    end;

```
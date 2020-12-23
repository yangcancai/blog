---
title: phoenix-guide
date: 2020-07-08 09:31:59
categories:
- elixir
tags:
- phoenix
---

## phoenix的流程

```elixir
 (crm_forward 0.1.0) lib/crm_forward_web/controllers/test_controller.ex:9: CrmForwardWeb.TestController.test_ping/2
        (crm_forward 0.1.0) lib/crm_forward_web/controllers/test_controller.ex:1: CrmForwardWeb.TestController.action/2
        (crm_forward 0.1.0) lib/crm_forward_web/controllers/test_controller.ex:1: CrmForwardWeb.TestController.phoenix_controller_pipeline/2
        (phoenix 1.5.6) lib/phoenix/router.ex:352: Phoenix.Router.__call__/2
        (crm_forward 0.1.0) lib/crm_forward_web/endpoint.ex:1: CrmForwardWeb.Endpoint.plug_builder_call/2
        (crm_forward 0.1.0) lib/plug/debugger.ex:132: CrmForwardWeb.Endpoint."call (overridable 3)"/2
        (crm_forward 0.1.0) lib/crm_forward_web/endpoint.ex:1: CrmForwardWeb.Endpoint.call/2
        (phoenix 1.5.6) lib/phoenix/endpoint/cowboy2_handler.ex:65: Phoenix.Endpoint.Cowboy2Handler.init/4
        (cowboy 2.8.0) /Users/admin/proj/elixir/crm/crm_forward/deps/cowboy/src/cowboy_handler.erl:37: :cowboy_handler.execute/2
        (cowboy 2.8.0) /Users/admin/proj/elixir/crm/crm_forward/deps/cowboy/src/cowboy_stream_h.erl:300: :cowboy_stream_h.execute/3
        (cowboy 2.8.0) /Users/admin/proj/elixir/crm/crm_forward/deps/cowboy/src/cowboy_stream_h.erl:291: :cowboy_stream_h.request_process/3
        (stdlib 3.12.1) proc_lib.erl:249: :proc_lib.init_p_do_apply/3
```
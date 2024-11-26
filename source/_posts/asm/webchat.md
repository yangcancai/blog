




-[MMComposeTextView keyDown:]:
回车发送后的关键指令
0000000101dafd30         mov        rdi, r13                                    ; End of try block started at 0x101dafd1b, Begin of try block (catch block at 0x101dafe5f)
0000000101dafd33         mov        rsi, r14
0000000101dafd36         call       qword [_objc_msgSend_105289770]             ; _objc_msgSend, _objc_msgSend_105289770
0000000101dafd3c         mov        rdi, rax                                    ; argument "instance" for method imp___stubs__objc_retainAutoreleasedReturnValue
0000000101dafd3f         call       imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
0000000101dafd44         mov        r12, rax                                    ; End of try block started at 0x101dafd30
0000000101dafd47         mov        rsi, qword [0x105ae5548]                    ; @selector(currentContentData)
0000000101dafd4e         mov        rdi, r13                                    ; Begin of try block (catch block at 0x101dafe3e)
                             ; NSKVONotifying_MMComposeTextView.currentContentData
                             ; 获取输入的内容列表 __NSArrayM的__NSCFString
0000000101dafd51         call       qword [_objc_msgSend_105289770]             ; _objc_msgSend, _objc_msgSend_105289770
0000000101dafd57         mov        rdi, rax                                    ; argument "instance" for method imp___stubs__objc_retainAutoreleasedReturnValue
0000000101dafd5a         call       imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
0000000101dafd5f         mov        r15, rax                                    ; End of try block started at 0x101dafd4e
0000000101dafd62         mov        rsi, qword [0x105b2ee48]                    ; @selector(textSenderInfo)
0000000101dafd69         mov        qword [rbp+var_C8], r13                     ; Begin of try block (catch block at 0x101dafe30)
0000000101dafd70         mov        rdi, r13
                             ; NSKVONotifying_MMComposeTextView.textSenderInfo
0000000101dafd73         call       qword [_objc_msgSend_105289770]             ; _objc_msgSend, _objc_msgSend_105289770
0000000101dafd79         mov        rdi, rax                                    ; argument "instance" for method imp___stubs__objc_retainAutoreleasedReturnValue
0000000101dafd7c         call       imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
0000000101dafd81         mov        r13, rax                                    ; End of try block started at 0x101dafd69
0000000101dafd84         mov        rsi, qword [0x105ae46c8]                    ; @selector(copy)
0000000101dafd8b         mov        rdi, rax                                    ; Begin of try block (catch block at 0x101dafe22)
0000000101dafd8e         call       qword [_objc_msgSend_105289770]    


p *(WeChat*)0x0000600001CFCB00
(WeChat) {
  _isLoggedIn = '\x01'
  _isLocked = '\0'
  _isInBackground = '\0'
  _isAppTerminating = '\0'
  _showDisconnectBanner = '\0'
  _isWCDBRepairing = '\0'
  _isClearing = '\0'
  _hasMsgBeforeSync = '\x01'
  _isWorkSpaceSleep = '\0'
  _isDisplayAsleep = '\0'
  _hasAuthOK = '\x01'
  _triedAutoLogin = '\0'
  _hasClearAndExitApp = '\0'
  _progress = 0
  _assertionID = 0
  _success = -536870911
  _mainWindowController = 0x0000600000eada80
  _chatsViewController = 0x00007fa73d031420
  _leftViewController = 0x0000600001991260
  _rightViewController = 0x00006000011f17c0
  _lockMenuItem = 0x0000600000be3480
  _wechatMenu = 0x000060000355bac0
  _scrollBarShowStatus = 1
  _locateMessage = 0
  _loginType = 1
  _ioMonitor = 0x000060000352a840
  _visualizationMonitor = 0x0000600000e84580
  _dockMenu = 0x0000600003532b00
  _safeModeWindowController = nil
  _shareExtConfirmWindowController = nil
  _serviceCenter = 0x00006000020c11a0
  _statusItem = 0x00006000035a0280
  _dockItem = 0x00006000020cbec0
  _feedbackItem = 0x0000600000bf0d20
  _logoutCGI = nil
  _diskMonitor = 0x0000600002067e00
  _focusMonitor = 0x0000600002031700
  _portsMonitor = nil
  _oomAlert = nil
  _flagView = nil
  _timer = nil
  _backupRecoverMenuItem = 0x0000600000bedf80
  _checkVersionHelper = 0x00006000022e90b0
}
 




 -[MMComposeTextView textView:shouldChangeTextInRange:replacementString:]:
 
 thread #1, queue = 'com.apple.main-thread', stop reason = instruction step over
    frame #0: 0x000000010d2d9ba4 WeChat`___lldb_unnamed_symbol132292 + 8 // -[MMComposeTextView textView:shouldChangeTextInRange:replacementString:]

rdi => 类实例地址
p (NSString*) 0x00007FA73F8A2400
(MMComposeTextView *) 0x00007fa73f8a2400 class name = NSKVONotifying_MMComposeTextView
rsi => 方法名称 
memory read 0x00007FF829AFAAF1 --format s
0x7ff829afaaf1: "textView:shouldChangeTextInRange:replacementString:"

rdx => MMComposeTextView 实例 
p *(MMComposeTextView*)0x00007FA73F8A2400
(MMComposeTextView) {
  MMRichAttachmentBaseTextView = {
    NSTextView = {
      NSText = {
        _ivars = nil
      }
    }
    _typingAttributesInProgress = '\0'
    _registerViewDictionary = 0x0000600002177ac0 3 key/value pairs
    _attachmentViews = 0x0000600002177ae0 0 elements
    _richAttachmentDelegate = 0x00007fa73d069c30
  }
  _canShowAlert = '\0'
  _hasJustInsertAt = '\0'
  _contentFromDraft = '\0'
  _hasSetSharpHightLight = '\0'
  _sendMsgShortCutType = 1
  _mmDelegate = 0x00007fa73d069dc0
  _mmEmotionDelegate = 0x00007fa73d069c30
  _textChangeDelegate = 0x00007fa73d069c30
  _composeInputViewDelegate = 0x00007fa73d069c30
  _referDelegate = 0x00007fa73d069c30
  _didClickSendBlock = 0x0000600002c9e130
  _didPasteFilesBlock = 0x0000600002c9e220
  _didDragFilesBlock = 0x0000600002c9e1c0
  _didPasteMessagesBlock = 0x0000600002c9e250
  _forwardMessageBlock = 0x0000600002c9e280
  _didConfirmForwardFavoritesItemBlock = 0x0000600002c9e2b0
  _didDragContactBlock = 0x0000600002c9e2e0
  _didDragWeAppBlock = 0x0000600002c9e1f0
  _didHandleEscapeKeyEventBlock = 0x0000600002c9e310
  _didHandleClearBlock = 0x0000600002c9e340
  _mentionTokenRanges = 0x0000600002c9c120 @"0 elements"
  _mentionUserList = 0x0000600002c9c0f0 @"0 elements"
  _chatUserName = 0x74ba6c7ba82f4ab1 @"filehelper"
  _mentionPopover = nil
  _mentionViewController = 0x0000600001593840
  _lastCheckTime = 746590250232
  _timeChecker = 0x000060000257b0a0
  _dragImgPath = nil
  _dragImgFolder = 0x00006000017bc370 @"/Users/vv/Library/Containers/com.tencent.xinWeChat/Data/Library/Caches/com.tencent.xinWeChat/2.0b4.0.9/ae5a894ec620c49083bcf45bd7164721/dragImgTmp"
  _editStyle = 4
  _textSenderInfo = nil
}
rcx => NSRange (将要被替换的文本范围。如果是插入操作，该范围的长度为 0)

r9 => 为输入的字符串
p (NSString*) 0x000060000321BC60
(__NSCFString *) 0x000060000321bc60 @"o"

    frame #1: 0x00007ff80d2d73ea AppKit`-[NSTextView(NSSharing) shouldChangeTextInRanges:replacementStrings:] + 1133
    frame #2: 0x00007ff80dbfb1b8 AppKit`-[NSTextView setMarkedText:selectedRange:replacementRange:] + 531
    frame #3: 0x00007ff80de76757 AppKit`-[NSTextInputContext(NSInputContext_WithCompletion) setMarkedText:selectedRange:replacementRange:completionHandler:] + 207
    frame #4: 0x00007ff80de7186b AppKit`__55-[NSTextInputContext handleTSMEvent:completionHandler:]_block_invoke.423 + 113
    frame #5: 0x00007ff80d2dbf03 AppKit`-[NSTextInputContext tryHandleTSMEvent_setMarkedText_withContext:dispatchCondition:setupForDispatch:dispatchWork:continuation:] + 99
    frame #6: 0x00007ff80de70f52 AppKit`__55-[NSTextInputContext handleTSMEvent:completionHandler:]_block_invoke.377 + 275
    frame #7: 0x00007ff80de70d8a AppKit`__55-[NSTextInputContext handleTSMEvent:completionHandler:]_block_invoke_2.371 + 224
    frame #8: 0x00007ff80de70344 AppKit`__55-[NSTextInputContext handleTSMEvent:completionHandler:]_block_invoke.321 + 2023
    frame #9: 0x00007ff80d2d53b8 AppKit`__55-[NSTextInputContext handleTSMEvent:completionHandler:]_block_invoke_2 + 74
    frame #10: 0x00007ff80d2d533f AppKit`-[NSTextInputContext tryHandleTSMEvent_HasMarkedText_withDispatchCondition:dispatchWork:continuation:] + 86
    frame #11: 0x00007ff80d2d49c6 AppKit`-[NSTextInputContext handleTSMEvent:completionHandler:] + 1850
    frame #12: 0x00007ff80d2d421d AppKit`_NSTSMEventHandler + 299
    frame #13: 0x00007ff814463573 HIToolbox`DispatchEventToHandlers(EventTargetRec*, OpaqueEventRef*, HandlerCallRec*) + 1364
    frame #14: 0x00007ff8144629bf HIToolbox`SendEventToEventTargetInternal(OpaqueEventRef*, OpaqueEventTargetRef*, HandlerCallRec*) + 336
    frame #15: 0x00007ff81446285e HIToolbox`SendEventToEventTargetWithOptions + 45
    frame #16: 0x00007ff8144bbbb1 HIToolbox`SendTSMEvent_WithCompletionHandler + 388
  * frame #17: 0x00007ff8146145c0 HIToolbox`__SendTextInputEvent_WithCompletionHandler_block_invoke + 511
    frame #18: 0x00007ff814612f08 HIToolbox`SendTextInputEvent_WithCompletionHandler + 1046
    frame #19: 0x00007ff8146751d1 HIToolbox`-[IMKInputSession _postEvent:completionHandler:] + 156
    frame #20: 0x00007ff8146870ab HIToolbox`__83-[IMKInputSession setMarkedText:selectionRange:replacementRange:completionHandler:]_block_invoke + 94
    frame #21: 0x00007ff814687e06 HIToolbox`__55-[IMKInputSession selectedRange_withCompletionHandler:]_block_invoke + 509
    frame #22: 0x00007ff8144d046d HIToolbox`invocation function for block in DispatchEventToHandlers(EventTargetRec*, OpaqueEventRef*, HandlerCallRec*) + 110
    frame #23: 0x00007ff80de736d6 AppKit`__55-[NSTextInputContext handleTSMEvent:completionHandler:]_block_invoke.567 + 97
    frame #24: 0x00007ff80d2d48e2 AppKit`-[NSTextInputContext handleTSMEvent:completionHandler:] + 1622
    frame #25: 0x00007ff80d2d421d AppKit`_NSTSMEventHandler + 299
    frame #26: 0x00007ff814463573 HIToolbox`DispatchEventToHandlers(EventTargetRec*, OpaqueEventRef*, HandlerCallRec*) + 1364
    frame #27: 0x00007ff8144629bf HIToolbox`SendEventToEventTargetInternal(OpaqueEventRef*, OpaqueEventTargetRef*, HandlerCallRec*) + 336
    frame #28: 0x00007ff81446285e HIToolbox`SendEventToEventTargetWithOptions + 45
    frame #29: 0x00007ff8144bbbb1 HIToolbox`SendTSMEvent_WithCompletionHandler + 388
    frame #30: 0x00007ff8146145c0 HIToolbox`__SendTextInputEvent_WithCompletionHandler_block_invoke + 511
    frame #31: 0x00007ff814612f08 HIToolbox`SendTextInputEvent_WithCompletionHandler + 1046
    frame #32: 0x00007ff8146751d1 HIToolbox`-[IMKInputSession _postEvent:completionHandler:] + 156
    frame #33: 0x00007ff814687963 HIToolbox`-[IMKInputSession selectedRange_withCompletionHandler:] + 285
    frame #34: 0x00007ff814672e1b HIToolbox`__72-[IMKInputSession _updateIMKMarkedRange:markedLength:completionHandler:]_block_invoke.981 + 74
    frame #35: 0x00007ff814672cc1 HIToolbox`-[IMKInputSession tryUpdateIMKMarkedRange_withDispatchCondition:dispatchWork:continuation:] + 87
    frame #36: 0x00007ff814672dac HIToolbox`-[IMKInputSession _updateIMKMarkedRange:markedLength:completionHandler:] + 174
    frame #37: 0x00007ff81468703e HIToolbox`-[IMKInputSession setMarkedText:selectionRange:replacementRange:completionHandler:] + 543
    frame #38: 0x00007ff81468755c HIToolbox`-[IMKInputSession setMarkedText:selectionRange:replacementRange:validFlags:completionHandler:] + 121
    frame #39: 0x00007ff8146786eb HIToolbox`__89-[IMKInputSession imkxpc_setMarkedText:selectionRange:replacementRange:validFlags:reply:]_block_invoke + 570
    frame #40: 0x00007ff814664c40 HIToolbox`__87+[IMKInputSession IMKXPCPerformBlockOnMainThreadInMode:performHow:callerCmd:workBlock:]_block_invoke + 103
    frame #41: 0x00007ff809a796ee CoreFoundation`__CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__ + 12
    frame #42: 0x00007ff809a79623 CoreFoundation`__CFRunLoopDoBlocks + 398
    frame #43: 0x00007ff809a78980 CoreFoundation`__CFRunLoopRun + 2182
    frame #44: 0x00007ff809a77b32 CoreFoundation`CFRunLoopRunSpecific + 557
    frame #45: 0x00007ff814664342 HIToolbox`-[IMKInputSessionXPCInvocation runPrivateRunloopWithTimeLimit:] + 102
    frame #46: 0x00007ff8144cc53f HIToolbox`-[IMKInputSessionXPCInvocation invocationAwaitXPCReply] + 235
    frame #47: 0x00007ff81466733b HIToolbox`__49-[IMKInputSession handleEvent:completionHandler:]_block_invoke_2.581 + 1547
    frame #48: 0x00007ff81466685f HIToolbox`__49-[IMKInputSession handleEvent:completionHandler:]_block_invoke_2.551 + 431
    frame #49: 0x00007ff814654468 HIToolbox`-[IMKClient switchedInputMode:completionHandler:] + 396
    frame #50: 0x00007ff8146658a8 HIToolbox`-[IMKInputSession tryHandleEventSwitchedInputMode:eventWasHandled:continuationHandler:] + 110
    frame #51: 0x00007ff814666675 HIToolbox`__49-[IMKInputSession handleEvent:completionHandler:]_block_invoke.550 + 190
    frame #52: 0x00007ff8146664c6 HIToolbox`__49-[IMKInputSession handleEvent:completionHandler:]_block_invoke + 255
    frame #53: 0x00007ff814671f7f HIToolbox`-[IMKInputSession _eventIsOn:completionHandler:] + 2163
    frame #54: 0x00007ff814665d6e HIToolbox`-[IMKInputSession handleEvent:completionHandler:] + 807
    frame #55: 0x00007ff81462146a HIToolbox`IMKInputSessionProcessEventRefWithCompletionHandler + 114
    frame #56: 0x00007ff814620c28 HIToolbox`InputMethodInstanceProcessEventRef_WithCompletionHandler + 122
    frame #57: 0x00007ff814612432 HIToolbox`__TSMEventToInputMethod_WithCompletionHandler_block_invoke + 117
    frame #58: 0x00007ff81461578c HIToolbox`__SendTSMDocumentLockEvent_WithCompletionHandler_block_invoke + 98
    frame #59: 0x00007ff8144d046d HIToolbox`invocation function for block in DispatchEventToHandlers(EventTargetRec*, OpaqueEventRef*, HandlerCallRec*) + 110
    frame #60: 0x00007ff80d2d485e AppKit`-[NSTextInputContext handleTSMEvent:completionHandler:] + 1490
    frame #61: 0x00007ff80d2d421d AppKit`_NSTSMEventHandler + 299
    frame #62: 0x00007ff814463573 HIToolbox`DispatchEventToHandlers(EventTargetRec*, OpaqueEventRef*, HandlerCallRec*) + 1364
    frame #63: 0x00007ff8144629bf HIToolbox`SendEventToEventTargetInternal(OpaqueEventRef*, OpaqueEventTargetRef*, HandlerCallRec*) + 336
    frame #64: 0x00007ff81446285e HIToolbox`SendEventToEventTargetWithOptions + 45
    frame #65: 0x00007ff8144bbbb1 HIToolbox`SendTSMEvent_WithCompletionHandler + 388
    frame #66: 0x00007ff814612376 HIToolbox`TrySendLockEvent_BeforeEventToInputMethod_WithContinuationHandler + 320
    frame #67: 0x00007ff8146121fc HIToolbox`TSMEventToInputMethod_WithCompletionHandler + 147
    frame #68: 0x00007ff814612149 HIToolbox`TSMEventToKeyboardInputMethod_WithCompletionHandler + 117
    frame #69: 0x00007ff8144bb115 HIToolbox`TSMKeyEvent_WithCompletionHandler + 574
    frame #70: 0x00007ff8144bf7e0 HIToolbox`__TSMProcessRawKeyEventWithOptionsAndCompletionHandler_block_invoke_5 + 251
    frame #71: 0x00007ff8144baecc HIToolbox`__TSMProcessRawKeyEventWithOptionsAndCompletionHandler_block_invoke_4 + 311
    frame #72: 0x00007ff8144bad03 HIToolbox`__TSMProcessRawKeyEventWithOptionsAndCompletionHandler_block_invoke_3 + 262
    frame #73: 0x00007ff8144baaba HIToolbox`__TSMProcessRawKeyEventWithOptionsAndCompletionHandler_block_invoke_2 + 283
    frame #74: 0x00007ff8144ba85c HIToolbox`__TSMProcessRawKeyEventWithOptionsAndCompletionHandler_block_invoke + 275
    frame #75: 0x00007ff8144a95fb HIToolbox`TSMProcessRawKeyEventWithOptionsAndCompletionHandler + 2345
    frame #76: 0x00007ff80de74a57 AppKit`__84-[NSTextInputContext _handleEvent:options:allowingSyntheticEvent:completionHandler:]_block_invoke_3.917 + 111
    frame #77: 0x00007ff80de747b6 AppKit`__204-[NSTextInputContext tryTSMProcessRawKeyEvent_orSubstitution:dispatchCondition:setupForDispatch:furtherCondition:doubleSpaceSubstitutionCondition:doubleSpaceSubstitutionWork:dispatchTSMWork:continuation:]_block_invoke.881 + 120
    frame #78: 0x00007ff80d2d2d52 AppKit`-[NSTextInputContext tryTSMProcessRawKeyEvent_orSubstitution:dispatchCondition:setupForDispatch:furtherCondition:doubleSpaceSubstitutionCondition:doubleSpaceSubstitutionWork:dispatchTSMWork:continuation:] + 246
    frame #79: 0x00007ff80d2d2750 AppKit`-[NSTextInputContext _handleEvent:options:allowingSyntheticEvent:completionHandler:] + 1396
    frame #80: 0x00007ff80d2d21a2 AppKit`-[NSTextInputContext _handleEvent:allowingSyntheticEvent:] + 105
    frame #81: 0x00007ff80d2d2018 AppKit`-[NSView interpretKeyEvents:] + 208
    frame #82: 0x00007ff80d2d1e65 AppKit`-[NSTextView keyDown:] + 722
    frame #83: 0x000000010d2d8bc9 WeChat`___lldb_unnamed_symbol132286 + 1710
    frame #84: 0x00007ff80d23f142 AppKit`-[NSWindow(NSEventRouting) _reallySendEvent:isDelayedEvent:] + 520
    frame #85: 0x00007ff80d23ed1f AppKit`-[NSWindow(NSEventRouting) sendEvent:] + 345
    frame #86: 0x00007ff80d9ee2b6 AppKit`-[NSApplication(NSEventRouting) sendEvent:] + 346
    frame #87: 0x000000010f41ca8e WeChat`___lldb_unnamed_symbol224458 + 61
    frame #88: 0x00007ff80d5a95c2 AppKit`-[NSApplication _handleEvent:] + 65
    frame #89: 0x00007ff80d0d102a AppKit`-[NSApplication run] + 640
    frame #90: 0x0000000113b35f4c libwmpf_host_export.dylib`___lldb_unnamed_symbol1112 + 3490108
    frame #91: 0x0000000113b34a7b libwmpf_host_export.dylib`___lldb_unnamed_symbol1112 + 3484779
    frame #92: 0x0000000113b0e135 libwmpf_host_export.dylib`___lldb_unnamed_symbol1112 + 3326757
    frame #93: 0x0000000113af3099 libwmpf_host_export.dylib`___lldb_unnamed_symbol1112 + 3216009
    frame #94: 0x00000001139f3151 libwmpf_host_export.dylib`___lldb_unnamed_symbol1112 + 2167617
    frame #95: 0x000000010bf18fcf WeChat`___lldb_unnamed_symbol48012 + 49
    frame #96: 0x000000010ce878d5 WeChat`___lldb_unnamed_symbol117980 + 1228
    frame #97: 0x00007ff809611366 dyld`start + 1942


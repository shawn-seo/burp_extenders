원래 주소 : http://drops.wooyun.org/tools/14685
0x00 서문
이 글은 "BurpSuite Plug-in 개발 가이드 API의 제 1 부" 이후에 작성되었습니다 . 이 글에서 소개 할 API 는 이전 의 API 와 밀접한 관련이 있으므로 독자는 다음과 전체를 전체적으로 취급 할 것을 권장합니다.

"BurpSuite 플러그인 개발 가이드"기사 시리즈는 다음과 같습니다.

"BurpSuite 플러그인 개발을위한 API"
"Java 용 BurpSuite 플러그인 개발 안내서"
"파이썬 용 BurpSuite 플러그인 개발 안내서"
0x01 API 참조
IMessageEditor
공용 인터페이스 IMessageEditor

이 인터페이스는 HTTP 메시지 편집 상자의 Burp 인스턴스를 사용하여 확장이 자체 UI에서 메시지 편집 상자를 사용할 수 있도록 확장을 제공하는 데 사용되며, 확장은 IBurpExtenderCallbacks.createMessageEditor () 를 호출 하여이 인터페이스의 인스턴스를 얻을 수 있습니다 .

이 인터페이스는 다음과 같은 방법을 제공합니다.

#!java
// 此方法返回了编辑器的 UI 组件，扩展插件可以将其添加到自己的 UI 中
java.awt.Component  getComponent()

// 此方法用于获取当前已显示的消息，此消息可能已被用户修改
byte[]  getMessage()

// 此方法返回了用户当前所选择的数据
byte[]  getSelectedData()

// 此方法用于决定当前的消息是否可被用户修改
boolean isMessageModified()

// 此方法用于将一个 HTTP 消息显示在编辑器中
void    setMessage(byte[] message, boolean isRequest)
참고 :이 인터페이스는 IMessageEditorTabFactory 와 같은 관련 인터페이스와 함께 사용해야 합니다.

데모 코드 ：

请见 IMessageEditorTabFactory 的实例代码。
IMessageEditorController
공용 인터페이스 IMessageEditorController

이 인터페이스는 IMessageEditor 에서 현재 표시된 메시지의 세부 사항 을 가져 오는 데 사용 됩니다. Burp의 HTTP 메시지 편집기 인스턴스를 생성하는 확장은 선택적으로 IMessageEditorController 인터페이스를 구현할 수 있습니다 . 확장 프로그램에 현재 메시지에 대한 추가 정보가 필요한 경우 편집기는이 인터페이스를 호출합니다 (예 : 현재 메시지를 다른 Burp 도구로 전송). ). 확장 은 IMessageEditorTabFactory 팩토리를 통해 사용자 정의 편집기 탭 페이지 를 제공합니다.이 팩토리의 createNewInstance 메소드는 팩토리가 생성 한 각 탭 페이지에 대한 IMessageEditorController 오브젝트에 대한 참조를 승인하며 탭 페이지에 현재 메시지에 대한 추가 정보가 필요할 때 호출됩니다. 객체.

이 방법은 다음과 같은 방법을 제공합니다.

#!java
// 此方法用于获取当前消息的 HTTP 服务信息
IHttpService    getHttpService()

// 此方法用于获取当前消息的 HTTP 请求（也有可能是一个响应消息）
byte[]  getRequest()

// 此方法用于获取当前消息的 HTTP 响应（也有可能是一个请求消息）
byte[]  getResponse()
데모 코드 ：

请见 IMessageEditorTabFactory 的实例代码。
IMessageEditorTab
공용 인터페이스 IMessageEditorTab

확장은 IMessageEditorTabFactory 팩토리를 등록합니다 .이 팩토리의 createNewInstance 는 현재 인터페이스의 인스턴스를 리턴하며 Burp는 ​​HTTP 메시지 편집기에서 사용자 정의 탭을 작성합니다.

이 인터페이스는 다음과 같은 방법을 제공합니다.

#!java
// 此方法返回当前显示的消息
byte[]  getMessage()

// 此方法用于获取当前已被用户选择的数据
byte[]  getSelectedData()

// 此方法返回自定义标签页的标题
java.lang.String    getTabCaption()

// 此方法返回自定义标签页内容的组件
java.awt.Component  getUiComponent()

// 此方法用于指示在显示一个新的 HTTP 消息时，是否启用自定义的标签页
boolean isEnabled(byte[] content, boolean isRequest)

// 此方法用于决定当前显示的消息是否可被用户修改
boolean isModified()

// 此方法可以显示一个新的消息或者清空已存在的消息
void    setMessage(byte[] content, boolean isRequest)
데모 코드 ：

请见 IMessageEditorTabFactory 的实例代码。
IMessageEditorTabFactory
공용 인터페이스 IMessageEditorTabFactory

扩展可以实现此接口，并且可以调用 IBurpExtenderCallbacks.registerMessageEditorTabFactory() 注册一个自定义的消息编辑器标签页的工厂。扩展插件可以在 Burp 的 HTTP 编辑器中渲染或编辑 HTTP 消息。

此接口提供了一个方法：

#!java
// Burp 将会对每一个 HTTP 消息编辑器调用一次此方法，此工厂必须返回一个新的 IMessageEditorTab 对象
IMessageEditorTab   createNewInstance(IMessageEditorController controller, boolean editable)
Demo code：

#!java
package burp;

import java.awt.Component;
import java.io.PrintWriter;

public class BurpExtender implements IBurpExtender, IMessageEditorTabFactory{

    public PrintWriter stdout;
    public IExtensionHelpers helpers;
    private IBurpExtenderCallbacks callbacks;

    @Override
    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){

        this.stdout = new PrintWriter(callbacks.getStdout(), true);
        this.callbacks = callbacks;
        this.helpers = callbacks.getHelpers();
        callbacks.setExtensionName("Her0in");
        callbacks.registerMessageEditorTabFactory(this);
    }

    @Override
    public IMessageEditorTab createNewInstance(
            IMessageEditorController controller, boolean editable) {
        // 返回 IMessageEditorTab 的实例
        return new iMessageEditorTab();
    }

    class iMessageEditorTab implements IMessageEditorTab{

        // 创建一个新的文本编辑器
        private ITextEditor iTextEditor = callbacks.createTextEditor();

        @Override
        public String getTabCaption() {
            // 设置消息编辑器标签页的标题
            return "测试 MessageEditorTab";
        }

        @Override
        public Component getUiComponent() {
            // 返回 iTextEditor 的组件信息，当然也可以放置其他的组件
            return iTextEditor.getComponent();
        }

        @Override
        public boolean isEnabled(byte[] content, boolean isRequest) {
            // 在显示一个新的 HTTP 消息时，启用自定义的标签页
            // 通过 content 和 isRequest 也可以对特定的消息进行设置
            return true;
        }

        @Override
        public void setMessage(byte[] content, boolean isRequest) {
            // 把请求消息里面的 data 参数进行 Base64 编码操作
            // 这里并未处理参数中没有 data 时的异常
            IParameter parameter = helpers.getRequestParameter(content, "data");
            stdout.println("data = " + parameter.getValue());
            iTextEditor.setText(helpers.stringToBytes(helpers.base64Encode(parameter.getValue())));
        }

        @Override
        public byte[] getMessage() {
            // 获取 iTextEditor 的文本
            return iTextEditor.getText();
        }

        @Override
        public boolean isModified() {
            // 允许用户修改当前的消息
            return true;
        }

        @Override
        public byte[] getSelectedData() {
            // 直接返回 iTextEditor 中选中的文本
            return iTextEditor.getSelectedText();
        }

    }
}
加载上述代码生成的插件后，会显示自定义的标签页和文本编辑器。

注意：官网提供的自定义消息编辑器的代码有误！

그림

IParameter
public interface IParameter

此接口用于操控 HTTP 请求参数，开发者通过此接口可以灵活的获取请求或响应里的参数。

#!java
// 此方法用于获取参数名称
java.lang.String    getName()

// 此方法用于获取在 HTTP 请求里面的最后一个参数的名称
int getNameEnd()

// 此方法用于获取在 HTTP 请求里面的第一个参数的名称
int getNameStart()

// 此方法用于获取参数类型，参数的类型在 IParameter 接口中有定义
byte    getType()

// 此方法用于获取参数的值
java.lang.String    getValue()

// 此方法用于获取最后一个参数的值
int getValueEnd()

// 此方法用于获取第一个参数的值
int getValueStart()
Demo code：

#!java
package burp;

import java.io.PrintWriter;
import java.util.List;

public class BurpExtender implements IBurpExtender, IHttpListener{

    public PrintWriter stdout;
    public IExtensionHelpers helpers;
    private IBurpExtenderCallbacks callbacks;

    @Override
    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){

        this.stdout = new PrintWriter(callbacks.getStdout(), true);
        this.callbacks = callbacks;
        this.helpers = callbacks.getHelpers();
        callbacks.setExtensionName("Her0in");
        callbacks.registerHttpListener(this);
    }

    @Override
    public void processHttpMessage(int toolFlag, boolean messageIsRequest,
            IHttpRequestResponse messageInfo) {
        // 获取请求中的参数
        if(messageIsRequest){
            IRequestInfo iRequestInfo = helpers.analyzeRequest(messageInfo);
            // 获取请求中的所有参数
            List<IParameter> iParameters = iRequestInfo.getParameters();
            for (IParameter iParameter : iParameters) {
                if(iParameter.getType() == IParameter.PARAM_URL)
                    stdout.println("参数：" + iParameter.getName() + " 在 URL中");
                    stdout.println("参数：" + iParameter.getName() + " 的值为：" + iParameter.getValue());
            }
        }

    }
}
加载上述代码生成的插件后，执行效果如下图所示：

그림

IProxyListener
public interface IProxyListener

扩展可以实现此接口，并且可以通过调用 IBurpExtenderCallbacks.registerProxyListener() 注册一个代理监听器。在代理工具处理了请求或响应后会通知此监听器。扩展插件通过注册这样一个监听器，对这些消息执行自定义的分析或修改操作。

此接口提供了一个很常用的方法：

#!java
// 当代理工具处理 HTTP 消息时则会调用此方法
void    processProxyMessage(boolean messageIsRequest, IInterceptedProxyMessage message)
Demo code：

#!java
package burp;

import java.io.PrintWriter;
public class BurpExtender implements IBurpExtender, IProxyListener{

    public PrintWriter stdout;
    public IExtensionHelpers helpers;
    private IBurpExtenderCallbacks callbacks;

    @Override
    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){

        this.stdout = new PrintWriter(callbacks.getStdout(), true);
        this.callbacks = callbacks;
        this.helpers = callbacks.getHelpers();
        callbacks.setExtensionName("Her0in");
        callbacks.registerProxyListener(this);
    }

    @Override
    public void processProxyMessage(boolean messageIsRequest,
            IInterceptedProxyMessage message) {
            // TODO here
    }
}
IRequestInfo
public interface IRequestInfo

此接口被用于获取一个 HTTP 请求的详细信息。扩展插件可以通过调用 IExtensionHelpers.analyzeRequest() 获得一个 IRequestInfo 对象。

此接口提供了以下方法：

#!java  
// 此方法用于获取 HTTP body 在请求消息中的起始偏移量
int getBodyOffset()

// 此方法用于获取请求消息的 HTTP 类型
byte    getContentType()

// 此方法用于获取请求中包含的 HTTP 头
java.util.List<java.lang.String>    getHeaders()

// 此方法用于获取请求的 HTTP 方法
java.lang.String    getMethod()

// 此方法用于获取请求中包含的参数
java.util.List<IParameter>  getParameters()

// 此方法用于获取请求中的 URL
java.net.URL    getUrl()
Demo code:

#!java
package burp;

import java.io.PrintWriter;

public class BurpExtender implements IBurpExtender, IHttpListener{

    public PrintWriter stdout;
    public IExtensionHelpers helpers;

    @Override
    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){

        this.stdout = new PrintWriter(callbacks.getStdout(), true);
        this.helpers = callbacks.getHelpers();
        callbacks.setExtensionName("Her0in");
        callbacks.registerHttpListener(this);
    }

    @Override
    public void processHttpMessage(int toolFlag, boolean messageIsRequest,
            IHttpRequestResponse messageInfo) {
        // 打印出请求的 Url 和 响应码
        if(messageIsRequest){
            stdout.println(helpers.bytesToString(messageInfo.getRequest()));
        }
        else{
            IResponseInfo responseInfo = helpers.analyzeResponse(messageInfo.getResponse());
            short statusCode = responseInfo.getStatusCode();
            stdout.printf("响应码 => %d\r\n", statusCode);
        }
    }
}
加载上述代码生成的插件后，执行效果如下图所示：

그림

IResponseInfo
public interface IResponseInfo

이 인터페이스는 HTTP 요청의 세부 사항을 가져 오는 데 사용됩니다. 확장은 IExtensionHelpers.AnalyzeResponse ()를 호출 하여 IResponseInfo 개체 를 얻을 수 있습니다 .

#!java
// 此方法用于获取 HTTP body 在响应消息中的起始偏移量
int getBodyOffset()

// 此方法用于获取响应消息中设置的 HTTP Cookie
java.util.List<ICookie> getCookies()

// 此方法用于获取包含在响应消息中的 HTTP 头
java.util.List<java.lang.String>    getHeaders()

// 此方法用于获取根据 HTTP 响应判断出的 MIME 类型
java.lang.String    getInferredMimeType()

// 此方法用于获取 HTTP 响应头中指示的 MIME 类型
java.lang.String    getStatedMimeType()

// 此方法用于获取 HTTP 状态码
short   getStatusCode()
데모 코드 :

#!java
package burp;

import java.io.PrintWriter;

public class BurpExtender implements IBurpExtender, IHttpListener{

    public PrintWriter stdout;
    public IExtensionHelpers helpers;

    @Override
    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){

        this.stdout = new PrintWriter(callbacks.getStdout(), true);
        this.helpers = callbacks.getHelpers();
        callbacks.setExtensionName("Her0in");
        callbacks.registerHttpListener(this);
    }

    @Override
    public void processHttpMessage(int toolFlag, boolean messageIsRequest,
            IHttpRequestResponse messageInfo) {
        // 打印出请求的 Url 和 响应码
        if(messageIsRequest){
            stdout.println(helpers.bytesToString(messageInfo.getRequest()));
        }
        else{
            IResponseInfo responseInfo = helpers.analyzeResponse(messageInfo.getResponse());
            short statusCode = responseInfo.getStatusCode();
            stdout.printf("响应码 => %d\r\n", statusCode);
        }
    }
}
iScan 이슈
공용 인터페이스 IScanIssue

이 인터페이스는 스캐너 도구로 스캔 한 문제의 세부 정보를 얻는 데 사용됩니다. 확장은 IScannerListener 를 등록 하거나 IBurpExtenderCallbacks.getScanIssues () 를 호출 하여 스캔 문제점의 세부 사항을 얻을 수 있습니다 . 확장 프로그램은 IScannerCheck 인터페이스 를 등록 하거나 IBurpExtenderCallbacks.addScanIssue () 메서드를 호출하여 검색 문제 를 사용자 정의 할 수도 있습니다. 현재 확장 프로그램은이 인터페이스의 구현을 제공해야합니다.

이 인터페이스는 다음과 같은 방법을 제공합니다.

#!java  
// 此方法返回扫描问题的信任等级
java.lang.String    getConfidence()

// 此方法返回生成扫描问题所对应的 HTTP 消息
IHttpRequestResponse[]  getHttpMessages()

// 此方法返回生成扫描问题所对应的 HTTP 服务信息
IHttpService    getHttpService()

// 此方法返回指定扫描问题类型的背景描述信息
java.lang.String    getIssueBackground()

// 此方法返回指定的扫描问题的详细信息
java.lang.String    getIssueDetail()

// 此方法返回扫描问题类型的名称
java.lang.String    getIssueName()

// 此方法返回扫描问题类型的数字标志符
int getIssueType()

// 此方法返回指定扫描问题的解决方式的背景描述信息
java.lang.String    getRemediationBackground()

// 此方法返回指定扫描问题的解决方式的背景详情
java.lang.String    getRemediationDetail()

// 此方法返回扫描问题的错误等级
java.lang.String    getSeverity()

// 此方法返回生成扫描问题对应的 URL 信息
java.net.URL    getUrl()
데모 코드 :

请见 IScannerListener 的实例代码。
IScannerCheck
공용 인터페이스 IScannerCheck

확장은이 인터페이스를 구현 한 다음 IBurpExtenderCallbacks.registerScannerCheck () 를 호출 하여 사용자 정의 스캐너 도구 검사기 를 등록 할 수 있습니다 . 트림은 검사자에게 "활성 스캔"또는 "수동 스캔"을 수행하도록 지시하고 문제가 감지 된 경우이를보고합니다.

#!java
// 当自定义的Scanner工具的检查器针对同一个 URL 路径报告了多个扫描问题时，Scanner 工具会调用此方法
int consolidateDuplicateIssues(IScanIssue existingIssue, IScanIssue newIssue)

// Scanner 工具会对每一个可插入的点执行“主动扫描”
java.util.List<IScanIssue>  doActiveScan(IHttpRequestResponse baseRequestResponse, IScannerInsertionPoint insertionPoint)

// Scanner 工具会对每一个可插入的点执行“被动扫描”
java.util.List<IScanIssue>  doPassiveScan(IHttpRequestResponse baseRequestResponse)
데모 코드 ：

#!java
package burp;

import java.io.PrintWriter;
import java.util.List;

public class BurpExtender implements IBurpExtender, IScannerCheck{

    public PrintWriter stdout;
    public IExtensionHelpers helpers;

    @Override
    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){

        this.stdout = new PrintWriter(callbacks.getStdout(), true);
        this.helpers = callbacks.getHelpers();
        callbacks.setExtensionName("Her0in");
        callbacks.registerScannerCheck(this);
    }

    @Override
    public List<IScanIssue> doPassiveScan(
            IHttpRequestResponse baseRequestResponse) {
        // TODO here
        return null;
    }

    @Override
    public List<IScanIssue> doActiveScan(
            IHttpRequestResponse baseRequestResponse,
            IScannerInsertionPoint insertionPoint) {
        // TODO here
        return null;
    }

    @Override
    public int consolidateDuplicateIssues(IScanIssue existingIssue,
            IScanIssue newIssue) {
        // TODO here
        return 0;
    }
}
IScannerInsertionPoint
공용 인터페이스의 IScannerInsertionPoint

이 인터페이스는 스캐너 도구 검사기 스캔을위한 삽입 지점을 정의하는 데 사용됩니다. 가입에 의해 확장 될 수 IScannerCheck가 하거나 가입하여,이 인터페이스의 인스턴스를 획득 IScannerInsertionPointProvider을 사용하는 주사 트림의 인스턴스를 작성.

#!java  
// 此方法用于使用指定的 payload 在插入点构建一个请求
byte[]  buildRequest(byte[] payload)

// 此方法返回插入点的基本值
java.lang.String    getBaseValue()

// 此方法返回插入点的名称
java.lang.String    getInsertionPointName()

// 此方法返回插入点的类型，插入点类型在IScannerInsertionPoint接口中定义
byte    getInsertionPointType()

// 当使用指定的 payload 替换到插入点时，此方法可以决定 payload 在请求中的偏移量
int[]   getPayloadOffsets(byte[] payload)
데모 코드 :

请见 IScannerCheck 的实例代码。  
IScannerInsertionPointProvider
공용 인터페이스의 IScannerInsertionPointProvider

확장은이 인터페이스를 구현하고 IBurpExtenderCallbacks.registerScannerInsertionPointProvider () 를 호출 하여 사용자 정의 스캔 삽입 점 에 대한 팩토리를 등록 할 수 있습니다 .

이 인터페이스는 다음과 같은 방법을 제공합니다.

#!java
// 当扫描请求为“主动扫描”时， Scanner 工具将会调用此方法，并且提供者应该提供一个自定义插入点的列表以便用于扫描
java.util.List<IScannerInsertionPoint>  getInsertionPoints(IHttpRequestResponse baseRequestResponse)
IScannerListener
공용 인터페이스의 IScannerListener

확장은 IBurpExtenderCallbacks.registerScannerListener () 를 호출 하여이 인터페이스를 구현 하고 스캐너 도구의 리스너 를 등록 할 수 있습니다 . 스캐너 도구는 새로운 문제가 감지되면이 리스너에 알립니다. 확장은 스캔 문제에 대한 사용자 정의 분석 및 로깅을 위해 이러한 리스너를 등록합니다.

이 인터페이스는 다음과 같은 방법을 제공합니다.

#!java
// 当一个新的扫描问题被添加到 Burp 的Scanner工具的扫描结果中时，此方法将被 Burp 调用
void    newScanIssue(IScanIssue issue)
데모 코드 ：

#!java  
package burp;

import java.io.PrintWriter;

public class BurpExtender implements IBurpExtender, IScannerListener{

    public PrintWriter stdout;
    public IExtensionHelpers helpers;

    @Override
    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){

        this.stdout = new PrintWriter(callbacks.getStdout(), true);
        this.helpers = callbacks.getHelpers();
        callbacks.setExtensionName("Her0in");
        callbacks.registerScannerListener(this);
    }

    @Override
    public void newScanIssue(IScanIssue issue) {
        // TODO Auto-generated method stub
        stdout.println("扫描到新的问题 :");
        stdout.println("url => " + issue.getUrl());     
        stdout.println("详情 => " + issue.getIssueDetail());  
    }
}
위 코드에서 생성 된 플러그인을로드 한 후 다음 그림에 실행 효과가 표시됩니다.

그림

IScanQueueItem
공용 인터페이스 IScanQueueItem

이 인터페이스는 Burp의 스캐너 도구에서 활성화 된 스캔 대기열 항목의 세부 정보를 가져 오는 데 사용됩니다. 확장은 IBurpExtenderCallbacks.doActiveScan () 을 호출 하여 스캔 큐 항목 에 대한 참조를 얻을 수 있습니다 .

#!java  
// 此方法可以取消扫描队列项目中的扫描状态
void    cancel()

// 获取扫描队列项目生成的问题的细节
IScanIssue[]    getIssues()

// 此方法返回扫描队列项目发生错误时的网络错误号
int getNumErrors()

// 此方法返回扫描队列项目的攻击插入点的数量
int getNumInsertionPoints()

// 此方法返回扫描队列项目已经发出的请求的数量
int getNumRequests()

// 此方法返回扫描队列项目中已经完成扫描的百分比
byte    getPercentageComplete()

// 此方法返回扫描队列项目的状态描述
java.lang.String    getStatus()
IScopeChangeListener
공용 인터페이스 IScopeChangeListener

확장은 IBurpExtenderCallbacks.registerScopeChangeListener () 를 호출 하여이 인터페이스를 구현 하고 대상 도구 아래에서 범위 변경 리스너를 등록 할 수 있습니다 . 이 인터페이스는 트림 대상 도구 아래의 범위가 변경되면 알립니다.

이 인터페이스는 다음과 같은 방법을 제공합니다.

#!java
// 当 Burp 的 Target 工具下的 scope 发生变化时，将会调用此方法。
void    scopeChanged()
데모 코드 :

#!java  
package burp;

import java.io.PrintWriter;

public class BurpExtender implements IBurpExtender, IScopeChangeListener{

    public PrintWriter stdout;
    public IExtensionHelpers helpers;

    @Override
    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){

        this.stdout = new PrintWriter(callbacks.getStdout(), true);
        this.helpers = callbacks.getHelpers();
        callbacks.setExtensionName("Her0in");
        callbacks.registerScopeChangeListener(this);
    }

    @Override
    public void scopeChanged() {
        // 手动添加或右键菜单添加目标到 scope 列表，就会执行此方法
        stdout.println("scope 有变化！");
    }
}
위 코드에서 생성 된 플러그인을로드 한 후 다음 그림에 실행 효과가 표시됩니다.

그림

이슈 처리
공용 인터페이스 ISessionHandlingAction

확장은 IBurpExtenderCallbacks.registerSessionHandlingAction () 을 호출 하여이 메소드를 구현 하고 사용자 정의 세션 조작 조치 를 등록 할 수 있습니다 . 등록 된 각 세션 작업 작업은 세션 작업 규칙의 UI에서 사용할 수 있으며 사용자는 세션 작업 동작의 규칙으로이 중 하나를 선택할 수 있습니다. 사용자는 오퍼레이션을 직접 호출하거나 매크로 정의에 따라 오퍼레이션을 호출하도록 선택할 수 있습니다.

이 인터페이스는 다음 메소드를 호출합니다.

#!java      
// 此方法由 Burp 调用获取会话操作行为的名称
java.lang.String    getActionName()

// 当会话操作行为被执行时会调用此方法
void    performAction(IHttpRequestResponse currentRequest, IHttpRequestResponse[] macroItems)
ITab
공용 인터페이스 ITab

이 인터페이스는 사용자 정의 탭에 사용됩니다. IBurpExtenderCallbacks.addSuiteTab () 메소드를 호출 하여 사용자 정의 탭을 Burp의 UI에 표시하십시오.

#!java  
// 此方法用于获取自定义标签的标题文本
java.lang.String    getTabCaption()

// Burp 调用此方法获取自定义标签页显示的组件
java.awt.Component  getUiComponent()
데모 코드 ：

#!java
package burp;

import java.awt.Component;
import java.io.PrintWriter;

import javax.swing.JButton;
import javax.swing.JPanel;
import javax.swing.SwingUtilities;

public class BurpExtender implements IBurpExtender, ITab{

    public PrintWriter stdout;
    public IExtensionHelpers helpers;

    private JPanel jPanel1;
    private JButton jButton1;

    @Override
    public void registerExtenderCallbacks(final IBurpExtenderCallbacks callbacks){

        this.stdout = new PrintWriter(callbacks.getStdout(), true);
        this.helpers = callbacks.getHelpers();
        callbacks.setExtensionName("Her0in");
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                 //创建一个 JPanel
                 jPanel1 = new JPanel();
                 jButton1 = new JButton("点我");

                 // 将按钮添加到面板中
                 jPanel1.add(jButton1);

                 //自定义的 UI 组件
                 callbacks.customizeUiComponent(jPanel1);
                 //将自定义的标签页添加到Burp UI 中
                 callbacks.addSuiteTab(BurpExtender.this);
            }
       });
    }

    @Override
    public String getTabCaption() {
        // 返回自定义标签页的标题
        return "Her0in";
    }

    @Override
    public Component getUiComponent() {
        // 返回自定义标签页中的面板的组件对象
        return jPanel1;
    }
}
위 코드에서 생성 된 플러그인을로드 한 후 다음 그림에 실행 효과가 표시됩니다.

그림

ITempFile
공용 인터페이스 ITempFile

이 인터페이스는 IBurpExtenderCallbacks.saveToTempFile () 을 호출 하여 생성 된 임시 파일 을 작동 하는 데 사용됩니다 .

#!java
// 删除临时文件，此方法已过时
void    delete()

// 此方法用于获取临时文件内容的缓冲区
byte[]  getBuffer()
ITextEditor
공용 인터페이스 ITextEditor

이 인터페이스는 Burp의 원본 텍스트 편집기를 확장하는 데 사용되며 확장은 IBurpExtenderCallbacks.createTextEditor () 를 호출 하여이 인터페이스의 인스턴스를 얻습니다.

#!java  
// 此方法返回用于扩展添加自定义的编辑器的 UI 组件
java.awt.Component  getComponent()

// 此方法用于获取当前的已选择的文本
byte[]  getSelectedText()

// 此方法用于获取用户在已显示的文本中选择的边界
int[]   getSelectionBounds()

// 此方法用于获取当前已显示的文本
byte[]  getText()

// 此方法用于指示用户是否对编辑器的内容做了修改
boolean isTextModified()

// 此方法用于决定当前的编辑器是否可编辑
void    setEditable(boolean editable)

// 此方法用于更新编辑器下边的搜索框的搜索表达式
void    setSearchExpression(java.lang.String expression)

// 此方法用于更新编辑器中当前已显示的文本
void    setText(byte[] text)
데모 코드 :

请见 IMessageEditorTabFactory 的实例代码。

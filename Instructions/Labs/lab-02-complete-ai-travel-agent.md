---
lab:
  title: '랩: Azure OpenAI 및 의미 체계 커널 SDK를 사용하여 AI 에이전트 개발'
  module: 'Module 01: Build your kernel'
---

# 랩: AI 여행 도우미 완성하기
# 학생용 랩 매뉴얼

이 랩에서는 의미 체계 커널 SDK를 사용하여 AI 여행 도우미를 완료합니다. LLM(대규모 언어 모델) 서비스를 위한 엔드포인트를 만들고, 의미 체계 커널 함수를 만들고, 의미 체계 커널 SDK의 자동 함수 호출 기능을 사용하여 사용자의 의도를 제공된 일부 사전 빌드된 플러그인을 포함한 적절한 플러그인으로 라우팅합니다. 또한 대화 기록을 사용하여 LLM에 컨텍스트를 제공하고 사용자가 대화를 계속할 수 있도록 합니다.

## 랩 시나리오

여러분은 고객을 위한 맞춤형 여행 환경을 만드는 것을 전문으로 하는 여행사의 개발자입니다. 고객이 여행 목적지에 대해 자세히 알아보고 여행을 위한 활동을 계획하는 데 도움이 되는 AI 여행 도우미를 만드는 임무를 맡았습니다. AI 여행 도우미는 통화 금액을 변환하고, 목적지 및 활동을 제안하고, 다양한 언어로 유용한 문구를 제공하고, 문구를 번역할 수 있어야 합니다. 또한 AI 여행 도우미는 대화 기록을 사용하여 사용자의 요청에 대한 상황별 관련 응답을 제공할 수 있어야 합니다.

## 목표

이 랩을 완료하면 다음을 달성할 수 있습니다.

* LLM(대규모 언어 모델) 서비스에 대한 엔드포인트 만들기
* 의미 체계 커널 개체 빌드
* 의미 체계 커널 SDK를 사용하여 프롬프트 실행
* 의미 체계 커널 함수 및 플러그 인 만들기
* 의미 체계 커널 SDK의 자동 함수 호출 기능 사용

## 랩 설정

### 필수 조건

연습을 완료하려면 시스템에 다음이 설치되어어 있어야 합니다.

* [Visual Studio Code](https://code.visualstudio.com)
* [최신 .NET 8.0 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
* Visual Studio Code용 [C# 확장](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

### 개발 환경 준비

이러한 연습을 위해 시작 프로젝트를 사용할 수 있습니다. 시작 프로젝트를 설정하려면 다음 단계를 따릅니다.

> [!IMPORTANT]
> .NET Framework 8.0과 C#용 VS Code 확장 및 NuGet 패키지 관리자가 설치되어 있어야 합니다.

1. 새 브라우저 창에 다음 URL을 붙여넣습니다.
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/02/Lab-02-Starter.zip`

1. 페이지의 오른쪽 위에 있는 <kbd>...</kbd> 단추를 클릭하여 zip 파일을 다운로드하거나 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>S</kbd>를 누릅니다.

1. 데스크톱의 폴더와 같이 쉽게 찾고 기억할 수 있는 위치에 Zip 파일의 콘텐츠를 추출합니다.

1. Visual Studio Code를 열고 **파일** > **폴더 열기**를 선택합니다.

1. 추출한 **시작** 폴더로 이동하여 **폴더 선택**을 선택합니다.

1. 코드 편집기에서 **Program.cs** 파일을 엽니다.

> [!NOTE]
> 폴더에 있는 모든 파일을 신뢰하는지 묻는 메시지가 표시되면 **예, 작성자를 신뢰합니다**를 선택합니다.

## 연습 1: 의미 체계 커널 SDK를 사용하여 플러그 인 만들기

이 연습에서는 LLM(대규모 언어 모델) 서비스에 대한 엔드포인트를 만듭니다. 의미 체계 커널 SDK는 이 엔드포인트를 사용하여 LLM에 연결하고 프롬프트를 실행합니다. 의미 체계 커널 SDK는 HuggingFace, OpenAI, Azure Open AI LLM을 지원합니다. 이 예제에서는 Azure Open AI를 사용합니다.

**예상 연습 완료 시간**: 10분

### 작업 1: Azure OpenAI 리소스 만들기

1. [https://portal.azure.com](https://portal.azure.com)으로 이동합니다.

1. 기본 설정을 사용하여 새 Azure OpenAI 리소스를 만듭니다.

    > [!NOTE]
    > Azure OpenAI 리소스가 이미 있는 경우 이 단계를 건너뛸 수 있습니다.

1. 리소스가 생성되면 **리소스로 이동**을 선택합니다.

1. **개요** 페이지에서 **Azure Foundry 포털로 이동**을 선택합니다.

1. **새 배포 만들기**를 선택한 다음 **기본 모델에서** 선택합니다.

1. 모델 목록에서 **gpt-35-turbo-16k**를 선택합니다.

1. **확인**을 선택합니다.

1. 배포의 이름을 입력하고 기본 옵션을 그대로 둡니다.

1. 배포가 완료되면 Azure Portal에서 Azure OpenAI 리소스로 다시 이동합니다.

1. **리소스 관리**에서 **키 및 엔드포인트**를 선택합니다.

    다음 작업에서 여기에 있는 데이터를 사용하여 커널을 빌드합니다. 키를 안전하게 비공개로 유지해야 합니다!

### 작업 2: 네이티브 플러그 인 만들기

이 작업에서는 기본 통화에서 대상 통화로 금액을 변환할 수 있는 네이티브 함수 플러그인을 만듭니다.

1. Visual Studio Code 프로젝트로 돌아갑니다.

1. **appsettings.json** 파일을 열고 Azure OpenAI Services 모델 ID, 엔드포인트 및 API 키로 값을 업데이트합니다.

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. **Plugins** 폴더에서 **CurrencyConverterPlugin.cs**라는 파일로 이동합니다.

1. **CurrencyConverterPlugin.cs** 파일에서 **환율을 가져오는 커널 함수 만들기** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Create a kernel function that gets the exchange rate
    [KernelFunction("convert_currency")]
    [Description("Converts an amount from one currency to another, for example USD to EUR")]
    public static decimal ConvertCurrency(decimal amount, string fromCurrency, string toCurrency)
    {
        decimal exchangeRate = GetExchangeRate(fromCurrency, toCurrency);
        return amount * exchangeRate;
    }
    ```

    이 코드에서는 **KernelFunction** 데코레이터를 사용하여 네이티브 함수를 선언합니다. 또한 **Description** 데코레이터를 사용하여 함수의 기능에 대한 설명을 추가합니다. 다음으로 지정된 금액을 한 통화에서 다른 통화로 변환하는 몇 가지 논리를 추가합니다.

1. **Program.cs** 파일을 엽니다.

1. **커널에 플러그인 추가** 주석 아래에 있는 환율 계산 플러그인을 가져옵니다.

    ```c#
    // Add plugins to the kernel
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();
    ```

    다음으로 플러그 인을 테스트해 보겠습니다.

1. **Program.cs** 파일을 마우스 오른쪽 단추로 클릭하고 "통합 터미널에서 열기"를 클릭합니다.

1. 터미널에서 `dotnet run`를 입력합니다. 

    통화를 변환하라는 프롬프트 요청을 입력합니다(예: "홍콩에서 10달러는 얼마인가요?").

    다음과 비슷한 결과가 나타나야 합니다.

    ```output
    Assistant: 10 USD is equivalent to 77.70 Hong Kong dollars (HKD).
    ```

## 연습 2: 핸들바 프롬프트 만들기

이 연습에서는 핸드바 프롬프트에서 함수를 생성합니다. 이 함수는 LLM에 사용자를 위한 여행 여정을 만들도록 요청합니다. 그럼 시작하겠습니다.

**예상 연습 완료 시간**: 10분

### 작업 1: 핸들바 프롬프트에서 함수 만들기

1. **Program.cs** 파일에 다음 `using` 지시문을 추가합니다.

    `using Microsoft.SemanticKernel.PromptTemplates.Handlebars;`

1. **핸들바 프롬프트 만들기** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Create a handlebars prompt
    string hbprompt = """
        <message role="system">Instructions: Before providing the user with a travel itinerary, ask how many days their trip is</message>
        <message role="user">I'm going to {{city}}. Can you create an itinerary for me?</message>
        <message role="assistant">Sure, how many days is your trip?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">
        """;
    ```

    이 코드에서는 핸들바 템플릿 형식을 사용하여 퓨샷 프롬프트를 만듭니다. 프롬프트는 여행 여정을 만들기 전에 모델이 사용자로부터 더 많은 정보를 검색하도록 안내합니다.

1. **핸들바 형식을 사용하여 프롬프트 템플릿 구성 만들기** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Create the prompt template config using handlebars format
    var templateFactory = new HandlebarsPromptTemplateFactory();
    var promptTemplateConfig = new PromptTemplateConfig()
    {
        Template = hbprompt,
        TemplateFormat = "handlebars",
        Name = "GetItinerary",
    };
    ```

    이 코드에서는 프롬프트에서 핸들바 템플릿 구성을 만듭니다. 이를 사용하여 플러그 인 함수를 만들 수 있습니다.

1. **프롬프트에서 플러그인 함수 만들기** 주석 아래에 다음 코드를 추가합니다. 

    ```c#
    // Create a plugin function from the prompt
    var promptFunction = kernel.CreateFunctionFromPrompt(promptTemplateConfig, templateFactory);
    var itineraryPlugin = kernel.CreatePluginFromFunctions("TravelItinerary", [promptFunction]);
    kernel.Plugins.Add(itineraryPlugin);
    ```

    그런 다음 프롬프트에 대한 플러그 인 함수를 만들어 커널에 추가합니다. 이제 함수를 호출할 준비가 되었습니다.

1. 터미널에 `dotnet run`을 입력하여 코드를 실행합니다.

    다음 입력을 사용하여 LLM에 여정을 생성하도록 프롬프트를 입력합니다.

    ```output
    Assistant: How may I help you?
    User: I'm going to Hong Kong, can you create an itinerary for me?
    Assistant: Sure! How many days will you be staying in Hong Kong?
    User: 10
    Assistant: Great! Here's a 10-day itinerary for your trip to Hong Kong:
    ...
    ```

    이제 AI 여행 도우미의 시작이 갖춰졌습니다. 프롬프트 및 플러그 인을 사용하여 더 많은 기능을 추가해 보겠습니다.

1.  **커널에 플러그인 추가** 주석 아래에 항공편 예약 플러그인을 추가합니다.

    ```c#
    // Add plugins to the kernel
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();
    kernel.ImportPluginFromType<FlightBookingPlugin>();
    ```

    이 플러그인은 모의 세부 정보가 포함된 **flights.json** 파일을 사용하여 항공편 예약을 시뮬레이션합니다. 다음으로 도우미에 몇 가지 추가 시스템 프롬프트를 추가합니다.

1.  **채팅에 시스템 메시지 추가** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Add system messages to the chat
    history.AddSystemMessage("The current date is 01/10/2025");
    history.AddSystemMessage("You are a helpful travel assistant.");
    history.AddSystemMessage("Before providing destination recommendations, ask the user about their budget.");
    ```

    이러한 프롬프트는 원활한 사용자 경험을 만들며, 항공편 예약 플러그인을 시뮬레이션하는 데 도움이 됩니다. 이제 코드를 테스트할 준비가 되었습니다.

1. 터미널에 `dotnet run`을 입력합니다.

    다음 프롬프트 중 일부를 입력해 보세요.

    ```output
    1. Can you give me some destination recommendations for Europe?
    2. I want to go to Barcelona, can you create an itinerary for me?
    3. How many Euros is 100 USD?
    4. Can you book me a flight to Barcelona?
    ```

    다른 입력을 시도하고 여행 도우미가 어떻게 응답하는지 확인합니다.

## 연습 3: 작업에 대한 사용자 동의 필요

이 연습에서는 도우미가 사용자를 대신하여 항공편을 예약하도록 허용하기 전에 사용자의 승인을 요청하는 필터 호출 기능을 추가합니다. 그럼 시작하겠습니다.

### 작업 1: 함수 호출 필터 만들기

1. **PermissionFilter.cs**라는 이름의 새 파일을 만듭니다.

1. 새 에서 다음 코드를 입력합니다.

    ```c#
    #pragma warning disable SKEXP0001 
    using Microsoft.SemanticKernel;
    
    public class PermissionFilter : IFunctionInvocationFilter
    {
        public async Task OnFunctionInvocationAsync(FunctionInvocationContext context, Func<FunctionInvocationContext, Task> next)
        {
            // Check the plugin and function names
            
            await next(context);
        }
    }
    ```

    >[!NOTE] 
    > 의미 체계 커널 SDK의 버전 1.30.0에서는 함수 필터가 변경될 수 있으며 경고 표시 안 됨이 필요합니다. 

    이 코드에서는 `IFunctionInvocationFilter` 인터페이스를 구현합니다. 이 `OnFunctionInvocationAsync` 메서드는 AI 도우미에서 함수를 호출할 때마다 항상 호출됩니다.

1. 다음 코드를 추가하여 함수가 `book_flight` 호출되는 시기를 검색합니다.

    ```c#
    // Check the plugin and function names
    if ((context.Function.PluginName == "FlightBookingPlugin" && context.Function.Name == "book_flight"))
    {
        // Request user approval

        // Proceed if approved
    }
    ```

    이 코드는 `FunctionInvocationContext`을(를) 사용하여 어떤 플러그 인과 함수가 호출되었는지 확인합니다.

1. 항공편을 예약하기 위한 사용자의 허가를 요청하려면 다음 논리를 추가합니다.

    ```c#
    // Request user approval
    Console.WriteLine("System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)");
    Console.Write("User: ");
    string shouldProceed = Console.ReadLine()!;

    // Proceed if approved
    if (shouldProceed != "Y")
    {
        context.Result = new FunctionResult(context.Result, "The operation was not approved by the user");
        return;
    }
    ```

1. **Program.cs** 파일로 이동합니다.

1. **커널에 필터 추가** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Add filters to the kernel
    kernel.FunctionInvocationFilters.Add(new PermissionFilter());
    ```

1. 터미널에 `dotnet run`을 입력합니다.

    항공 예약하라는 프롬프트를 입력합니다. 다음과 유사한 응답이 표시됩니다.

    ```output
    User: Find me a flight to Tokyo on the 19
    Assistant: I found a flight to Tokyo on the 19th of January. The flight is with Air Japan and the price is $1200.
    User: Y
    System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)
    User: N
    Assistant: I'm sorry, but I am unable to book the flight for you.
    ```

    도우미는 예약을 진행하기 전에 사용자의 승인을 받아야 합니다.

### 검토

이 랩에서는 LLM(대규모 언어 모델) 서비스에 대한 엔드포인트를 만들고, 의미 체계 커널 개체를 빌드하고, 의미 체계 커널 SDK를 사용하여 프롬프트를 실행했습니다. 또한, 모델을 안내하기 위해 플러그인을 만들고 시스템 메시지를 활용했습니다. 축하합니다. 랩을 완료했습니다.

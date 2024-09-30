---
lab:
  title: '랩: Azure OpenAI 및 의미 체계 커널 SDK를 사용하여 AI 에이전트 개발'
  module: 'Module 01: Build your kernel'
---

# 랩: AI 여행사 완료
# 학생용 랩 매뉴얼

이 랩에서는 의미 체계 커널 SDK를 사용하여 AI 여행사를 완료합니다. LLM(대규모 언어 모델) 서비스를 위한 엔드포인트를 만들고, 의미 체계 커널 함수를 만들고, 의미 체계 커널 SDK의 자동 함수 호출 기능을 사용하여 사용자의 의도를 제공된 일부 사전 빌드된 플러그인을 포함한 적절한 플러그인으로 라우팅합니다. 또한 대화 기록을 사용하여 LLM에 컨텍스트를 제공하고 사용자가 대화를 계속할 수 있도록 합니다.

## 랩 시나리오

여러분은 고객을 위한 맞춤형 여행 환경을 만드는 것을 전문으로 하는 여행사의 개발자입니다. 고객이 여행 목적지에 대해 자세히 알아보고 여행을 위한 활동을 계획하는 데 도움이 되는 AI 여행사를 만드는 임무를 맡았습니다. AI 여행사는 통화 금액을 변환하고, 목적지 및 활동을 제안하고, 다양한 언어로 유용한 문구를 제공하고, 문구를 번역할 수 있어야 합니다. 또한 AI 여행사는 대화 기록을 사용하여 사용자의 요청에 대한 상황별 관련 응답을 제공할 수 있어야 합니다.

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

1. **개요** 페이지에서 **Azure OpenAI Studio로 이동**을 선택합니다.

:::image type="content" source="../media/model-deployments.png" alt-text="Azure OpenAI 배포 페이지의 스크린샷.":::

1. **새 배포 만들기**를 선택한 다음 **+새 배포 만들기**를 선택합니다.

1. **모델 배포** 팝업에서 **gpt-35-turbo-16k**를 선택합니다.

    기본 모델 버전 사용

1. 배포 이름 입력

1. 배포가 완료되면 Azure OpenAI 리소스로 다시 이동합니다.

1. **리소스 관리**에서 **키 및 엔드포인트**를 선택합니다.

    다음 작업에서 이러한 값을 사용하여 커널을 빌드합니다. 키를 비공개로 안전하게 유지해야 합니다!

1. Visual Studio Code의 **Program.cs** 파일로 돌아갑니다.

1. Azure Open AI Services 배포 이름, API 키, 엔드포인트로 다음 변수 업데이트

    ```csharp
    string yourDeploymentName = "";
    string yourEndpoint = "";
    string yourApiKey = "";
    ```

    > [!NOTE]
    > 일부 의미 체계 커널 SDK 기능이 작동하려면 배포 모델이 "gpt-35-turbo-16k"여야 합니다.

### 작업 2: 네이티브 함수 만들기

이 작업에서는 기본 통화에서 대상 통화로 금액을 변환할 수 있는 네이티브 함수를 만듭니다.

1. **Plugins/ConvertCurrency** 폴더에 **CurrencyConverter.cs**라는 새 파일 만들기

1. **CurrencyConverter.cs** 파일에서 다음 코드를 추가하여 플러그 인 함수를 만듭니다.

    ```c#
    using AITravelAgent;
    using System.ComponentModel;
    using Microsoft.SemanticKernel;

    class CurrencyConverter
    {
        [KernelFunction, 
        Description("Convert an amount from one currency to another")]
        public static string ConvertAmount()
        {
            var currencyDictionary = Currency.Currencies;
        }
    }
    ```

    이 코드에서는 **KernelFunction** 데코레이터를 사용하여 네이티브 함수를 선언합니다. 또한 **Description** 데코레이터를 사용하여 함수의 기능에 대한 설명을 추가합니다. **Currency.Currencies**를 사용하여 통화 및 환율 사전을 가져올 수 있습니다. 다음으로 지정된 금액을 한 통화에서 다른 통화로 변환하는 몇 가지 논리를 추가합니다.

1. **ConvertAmount** 함수를 다음 코드로 수정합니다.

    ```c#
    [KernelFunction, Description("Convert an amount from one currency to another")]
    public static string ConvertAmount(
        [Description("The target currency code")] string targetCurrencyCode, 
        [Description("The amount to convert")] string amount, 
        [Description("The starting currency code")] string baseCurrencyCode)
    {
        var currencyDictionary = Currency.Currencies;
        
        Currency targetCurrency = currencyDictionary[targetCurrencyCode];
        Currency baseCurrency = currencyDictionary[baseCurrencyCode];

        if (targetCurrency == null)
        {
            return targetCurrencyCode + " was not found";
        }
        else if (baseCurrency == null)
        {
            return baseCurrencyCode + " was not found";
        }
        else
        {
            double amountInUSD = Double.Parse(amount) * baseCurrency.USDPerUnit;
            double result = amountInUSD * targetCurrency.UnitsPerUSD;
            return @$"${amount} {baseCurrencyCode} is approximately 
                {result.ToString("C")} in {targetCurrency.Name}s ({targetCurrencyCode})";
        }
    }
    ```

    이 코드에서는 **Currency.Currencies** 사전을 사용하여 대상 및 기본 통화에 대한 **Currency** 개체를 가져옵니다. 그런 다음 **Currency** 개체를 사용하여 기본 통화에서 대상 통화로 금액을 변환합니다. 마지막으로, 변환된 양으로 문자열을 반환합니다. 다음으로 플러그 인을 테스트해 보겠습니다.

1. **Program.cs** 파일에서 다음 코드를 사용하여 새 플러그 인 함수를 가져오고 호출합니다.

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var result = await kernel.InvokeAsync("CurrencyConverter", 
        "ConvertAmount", 
        new() {
            {"targetCurrencyCode", "USD"}, 
            {"amount", "52000"}, 
            {"baseCurrencyCode", "VND"}
        }
    );

    Console.WriteLine(result);
    ```

    이 코드에서는 **ImportPluginFromType** 메서드를 사용하여 플러그 인을 가져옵니다. 그런 다음 **InvokeAsync** 메서드를 사용하여 플러그 인 함수를 호출합니다. **InvokeAsync** 메서드는 플러그 인 이름, 함수 이름 및 매개 변수 사전을 사용합니다. 마지막으로 결과를 콘솔에 출력합니다. 다음으로 코드를 실행하여 작동하는지 확인합니다.

1. 터미널 > 새 터미널을 선택하여 터미널을 엽니다.

1. 터미널에서 `dotnet run`를 입력합니다. 다음과 같은 출력이 표시됩니다.

    ```output
    $52000 VND is approximately $2.13 in US Dollars (USD)
    ```

    플러그 인이 제대로 작동했으므로 사용자가 변환하려는 통화와 금액을 감지할 수 있는 자연어 프롬프트를 만들어 보겠습니다.

### 작업 3: 프롬프트를 사용하여 사용자 입력 구문 분석

이 작업에서는 사용자의 입력을 구문 분석하여 변환할 대상 통화, 기본 통화 및 금액을 식별하는 프롬프트를 만듭니다.

1. **Prompts** 폴더에 **GetTargetCurrencies**라는 새 폴더 만들기

1. **GetTargetCurrencies** 폴더에서 **config.json**이라는 새 파일을 만들기

1. **config.json** 파일에 다음 텍스트를 입력합니다.

    ```output
    {
        "schema": 1,
        "type": "completion",
        "description": "Identify the target currency, base currency, and amount to convert",
        "execution_settings": {
            "default": {
                "max_tokens": 800,
                "temperature": 0
            }
        },
        "input_variables": [
            {
                "name": "input",
                "description": "Text describing some currency amount to convert",
                "required": true
            }
        ]
    }
    ```

1. **GetTargetCurrencies** 폴더에서 **skprompt.txt**라는 새 파일을 만들기

1. **skprompt.txt** 파일에 다음 텍스트를 입력합니다.

    ```html
    <message role="system">Identify the target currency, base currency, and 
    amount from the user's input in the format target|base|amount</message>

    For example: 

    <message role="user">How much in GBP is 750.000 VND?</message>
    <message role="assistant">GBP|VND|750000</message>

    <message role="user">How much is 60 USD in New Zealand Dollars?</message>
    <message role="assistant">NZD|USD|60</message>

    <message role="user">How many Korean Won is 33,000 yen?</message>
    <message role="assistant">KRW|JPY|33000</message>

    <message role="user">{{$input}}</message>
    <message role="assistant">target|base|amount</message>
    ```

### 작업 4: 작업 확인

이 작업에서는 애플리케이션을 실행하고 코드가 올바르게 작동하는지 확인합니다. 

1. 다음 코드로 **Program.cs** 파일을 업데이트하여 새 프롬프트를 테스트합니다.

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var result = await kernel.InvokeAsync(prompts["GetTargetCurrencies"],
        new() {
            {"input", "How many Australian Dollars is 140,000 Korean Won worth?"}
        }
    );

    Console.WriteLine(result);
    ```

1. 터미널에 `dotnet run`을 입력합니다. 다음과 같은 출력이 표시됩니다.

    ```output
    AUD|KRW|140000
    ```

    > [!NOTE]
    > 코드가 예상한 결과를 생성하지 못하는 경우 **솔루션** 폴더에서 코드를 검토할 수 있습니다. 더 정확한 결과를 생성하려면 **skprompt.txt** 파일에서 프롬프트를 조정해야 할 수 있습니다.

이제 한 통화에서 다른 통화로 금액을 변환할 수 있는 플러그 인과 사용자의 입력을 **ConvertAmount** 함수에서 사용할 수 있는 형식으로 구문 분석하는 데 사용할 수 있는 프롬프트가 있습니다. 이렇게 하면 사용자가 AI 여행사를 사용하여 통화 금액을 쉽게 변환할 수 있습니다.

## 연습 2: 사용자 의도에 따라 플러그 인 선택 자동화

이 연습에서는 사용자의 의도를 검색하고 대화를 원하는 플러그 인으로 라우팅합니다. 제공된 플러그 인을 사용하여 사용자의 의도를 검색할 수 있습니다. 그럼 시작하겠습니다.

**예상 연습 완료 시간**: 10분

### 작업 1: GetIntent 플러그 인 사용

1. **Program.cs** 파일을 다음 코드로 업데이트합니다.

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    Console.WriteLine("What would you like to do?");
    var input = Console.ReadLine();

    var intent = await kernel.InvokeAsync<string>(
        prompts["GetIntent"], 
        new() {{ "input",  input }}
    );

    ```

    이 코드에서는 **GetIntent** 프롬프트를 사용하여 사용자의 의도를 검색합니다. 그런 다음 **intent**라는 변수에 의도를 저장합니다. 다음으로 의도를 **CurrencyConverter** 플러그 인으로 라우팅합니다.

1. `Program.cs` 파일에 다음 코드를 추가합니다.

    ```c#
    switch (intent) {
        case "ConvertCurrency": 
            var currencyText = await kernel.InvokeAsync<string>(
                prompts["GetTargetCurrencies"], 
                new() {{ "input",  input }}
            );
            var currencyInfo = currencyText!.Split("|");
            var result = await kernel.InvokeAsync("CurrencyConverter", 
                "ConvertAmount", 
                new() {
                    {"targetCurrencyCode", currencyInfo[0]}, 
                    {"baseCurrencyCode", currencyInfo[1]},
                    {"amount", currencyInfo[2]}, 
                }
            );
            Console.WriteLine(result);
            break;
        default:
            Console.WriteLine("Other intent detected");
            break;
    }
    ```

    **GetIntent** 플러그 인은 다음 값을 반환합니다. ConvertCurrency, SuggestDestinations, SuggestActivities, Translate, HelpfulPhrases, Unknown. **switch** 문을 사용하여 사용자의 의도를 적절한 플러그 인으로 라우팅합니다. 
    
    사용자의 의도가 통화를 환산하는 것이라면 **GetTargetCurrencies** 프롬프트를 사용하여 통화 정보를 검색합니다. 그런 다음 **CurrencyConverter** 플러그 인을 사용하여 금액을 변환합니다.

    다음으로 다른 의도를 처리하기 위해 몇 가지 사례를 추가합니다. 지금은 의미 체계 커널 SDK의 자동 함수 호출 기능을 사용하여 의도를 사용 가능한 플러그 인으로 라우팅해 보겠습니다.

1. **Program.cs** 파일에 다음 코드를 추가하여 자동 함수 호출 설정을 만듭니다.

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    OpenAIPromptExecutionSettings settings = new()
    {
        ToolCallBehavior = ToolCallBehavior.AutoInvokeKernelFunctions
    };

    Console.WriteLine("What would you like to do?");
    var input = Console.ReadLine();
    var intent = await kernel.InvokeAsync<string>(
        prompts["GetIntent"], 
        new() {{ "input",  input }}
    );
    ```

    다음으로 다른 의도에 대한 스위치 문에 사례를 추가합니다.

1. **Program.cs** 파일을 다음 코드로 업데이트합니다.

    ```c#
    switch (intent) {
        case "ConvertCurrency": 
            // ...Code you entered previously...
            break;
        case "SuggestDestinations":
        case "SuggestActivities":
        case "HelpfulPhrases":
        case "Translate":
            var autoInvokeResult = await kernel.InvokePromptAsync(input!, new(settings));
            Console.WriteLine(autoInvokeResult);
            break;
        default:
            Console.WriteLine("Other intent detected");
            break;
    }
    ```

    이 코드에서는 **AutoInvokeKernelFunctions** 설정을 사용하여 커널에서 참조되는 함수와 프롬프트를 자동으로 호출합니다. 사용자의 의도가 통화 환산이라면 **CurrencyConverter** 플러그 인이 해당 작업을 수행합니다. 
    
    사용자의 의도가 대상 또는 작업 제안을 가져오거나, 구를 번역하거나, 특정 언어로 유용한 구를 가져오는 것인 경우, **AutoInvokeKernelFunctions** 설정은 프로젝트 코드에 포함된 기존 플러그 인을 자동으로 호출합니다.

    이러한 의도 사례에 속하지 않는 경우 사용자 입력을 LLM(대규모 언어 모델)에 대한 프롬프트로 실행하는 코드를 추가할 수도 있습니다.

1. 다음 코드를 사용하여 기본 사례를 업데이트합니다.

    ```c#
    default:
        Console.WriteLine("Sure, I can help with that.");
        var otherIntentResult = await kernel.InvokePromptAsync(input!, new(settings));
        Console.WriteLine(otherIntentResult);
        break;
    ```

    이제 사용자가 다른 의도를 갖고 있는 경우 LLM이 사용자의 요청을 처리할 수 있습니다. 시도해 보기:

### 작업 2: 작업 확인

이 작업에서는 애플리케이션을 실행하고 코드가 올바르게 작동하는지 확인합니다. 

1. 터미널에 `dotnet run`을 입력합니다. 메시지가 표시되면 다음 프롬프트와 유사한 텍스트를 입력합니다.

    ```output
    What would you like to do?
    How many TTD is 50 Qatari Riyals?    
    ```

1. 다음 응답과 유사한 출력이 표시됩니다.

    ```output
    $50 QAR is approximately $93.10 in Trinidadian Dollars (TTD)
    ```

1. 터미널에 `dotnet run`을 입력합니다. 메시지가 표시되면 다음 프롬프트와 유사한 텍스트를 입력합니다.

    ```output
    What would you like to do?
    I want to go somewhere that has lots of warm sunny beaches and delicious, spicy food!
    ```

1. 다음 응답과 유사한 출력이 표시됩니다.

    ```output
    Based on your preferences for warm sunny beaches and delicious, spicy food, I have a few destination recommendations for you:

    1. Thailand: Known for its stunning beaches, Thailand offers a perfect combination of relaxation and adventure. You can visit popular beach destinations like Phuket, Krabi, or Koh Samui, where you'll find crystal-clear waters and white sandy shores. Thai cuisine is famous for its spiciness, so you'll have plenty of mouthwatering options to try, such as Tom Yum soup, Pad Thai, and Green Curry.

    2. Mexico: Mexico is renowned for its beautiful coastal regions and vibrant culture. You can explore destinations like Cancun, Playa del Carmen, or Tulum, which boast stunning beaches along the Caribbean Sea. Mexican cuisine is rich in flavors and spices, offering a wide variety of dishes like tacos, enchiladas, and mole sauces that will satisfy your craving for spicy food.

    ...

    These destinations offer a perfect blend of warm sunny beaches and delicious, spicy food, ensuring a memorable trip for you. Let me know if you need any further assistance or if you have any specific preferences for your trip!
    ```

1. 터미널에 `dotnet run`을 입력합니다. 메시지가 표시되면 다음 프롬프트와 유사한 텍스트를 입력합니다.

    ```output
    What would you like to do?
    Can you give me a recipe for chicken satay?

1. You should see a response similar to the following response:

    ```output
    Sure, I can help with that.
    Certainly! Here's a recipe for chicken satay:

    ...
    ```

    의도는 기본 사례로 라우팅되어야 하며 LLM은 치킨 사테 레시피 요청을 처리해야 합니다.

    > [!NOTE]
    > 코드가 예상한 결과를 생성하지 못하는 경우 **Solution** 폴더에서 코드를 검토할 수 있습니다.

다음으로 특정 플러그 인에 일부 대화 기록을 제공하도록 라우팅 논리를 수정해 보겠습니다. 기록을 제공하면 플러그 인이 사용자 요청에 대해 보다 상황에 맞는 응답을 검색할 수 있습니다.

### 작업 3: 전체 플러그 인 라우팅

이 연습에서는 대화 기록을 사용하여 LLM(대규모 언어 모델)에 컨텍스트를 제공합니다. 또한 실제 챗봇처럼 사용자가 대화를 계속할 수 있도록 코드를 조정합니다. 그럼 시작하겠습니다.

1. 사용자의 입력을 받아들이기 위해 do-while 루프를 사용하도록 코드를 수정합니다.

    ```c#
    string input;

    do 
    {
        Console.WriteLine("What would you like to do?");
        input = Console.ReadLine();

        // ...
    }
    while (!string.IsNullOrWhiteSpace(input));
    ```

    이제 사용자가 빈 줄을 입력할 때까지 대화를 계속할 수 있습니다.

1. **SuggestDestinations** 사례를 수정하여 사용자 여행에 대한 세부 정보를 캡처합니다.

    ```c#
    case "SuggestDestinations":
        chatHistory.AppendLine("User:" + input);
        var recommendations = await kernel.InvokePromptAsync(input!);
        Console.WriteLine(recommendations);
        break;
    ```

1. 다음 코드와 함께 **SuggestActivities** 사례의 여행 세부 정보를 사용합니다.

    ```c#
     case "SuggestActivities":
        var chatSummary = await kernel.InvokeAsync(
            "ConversationSummaryPlugin", 
            "SummarizeConversation", 
            new() {{ "input", chatHistory.ToString() }});
        break;
    ```

    이 코드에서는 기본 제공된 **SummarizeConversation** 함수를 사용하여 사용자와의 채팅을 요약합니다. 다음으로 요약을 활용하여 대상에서의 작업을 제안해 보겠습니다.

1. 다음 코드를 사용하여 **SuggestActivities** 사례를 확장합니다.

    ```c#
    var activities = await kernel.InvokePromptAsync(
        input,
        new () {
            {"input", input},
            {"history", chatSummary},
            {"ToolCallBehavior", ToolCallBehavior.AutoInvokeKernelFunctions}
    });

    chatHistory.AppendLine("User:" + input);
    chatHistory.AppendLine("Assistant:" + activities.ToString());
    
    Console.WriteLine(activities);
    break;
    ```

    이 코드에서는 **input** 및 **chatSummary**를 커널 인수로 추가합니다. 그런 다음 커널은 프롬프트를 호출하고 이를 **SuggestActivities** 플러그 인으로 라우팅합니다. 또한 사용자의 입력과 도우미의 응답을 채팅 기록에 추가하고 결과를 표시합니다. 다음으로, **SuggestActivities** 플러그 인에 **chatSummary** 변수를 추가해야 합니다.

1. **Prompts/SuggestActivities/config.json**으로 이동하여 Visual Studio Code에서 파일을 엽니다.

1. **input_variables** 아래에 채팅 기록에 대한 변수를 추가합니다.

    ```json
    "input_variables": [
      {
          "name": "history",
          "description": "Some background information about the user",
          "required": false
      },
      {
          "name": "destination",
          "description": "The destination a user wants to visit",
          "required": true
      }
   ]
   ```

1. **Prompts/SuggestActivities/skprompt.txt**로 이동하여 파일을 엽니다.

1. 프롬프트의 처음 절반을 채팅 기록 변수를 사용하는 다음 프롬프트로 바꿉니다.

    ```html 
    You are an experienced travel agent. 
    You are helpful, creative, and very friendly. 
    Consider the traveler's background: {{$history}}
    ```

    프롬프트의 나머지 부분은 그대로 둡니다. 이제 플러그 인은 채팅 기록을 사용하여 LLM에 컨텍스트를 제공합니다.

### 작업 4: 작업 확인

이 작업에서는 애플리케이션을 실행하고 코드가 올바르게 작동하는지 확인합니다.

1. 업데이트된 스위치 사례를 다음 코드와 비교합니다.

    ```c#
    case "SuggestDestinations":
            chatHistory.AppendLine("User:" + input);
            var recommendations = await kernel.InvokePromptAsync(input!);
            Console.WriteLine(recommendations);
            break;
    case "SuggestActivities":

        var chatSummary = await kernel.InvokeAsync(
            "ConversationSummaryPlugin", 
            "SummarizeConversation", 
            new() {{ "input", chatHistory.ToString() }});

        var activities = await kernel.InvokePromptAsync(
            input!,
            new () {
                {"input", input},
                {"history", chatSummary},
                {"ToolCallBehavior", ToolCallBehavior.AutoInvokeKernelFunctions}
        });

        chatHistory.AppendLine("User:" + input);
        chatHistory.AppendLine("Assistant:" + activities.ToString());
        
        Console.WriteLine(activities);
        break;
    ```

1. 터미널에 `dotnet run`을 입력합니다. 메시지가 표시되면 다음과 유사한 텍스트를 입력합니다.

    ```output
    What would you like to do?
    How much is 60 USD in new zealand dollars?
    ```

1. 다음과 유사한 출력이 표시됩니다.

    ```output
    $60 USD is approximately $97.88 in New Zealand Dollars (NZD)
    What would you like to do?
    ```

1. 몇 가지 상황 신호와 함께 대상 제안에 대한 프롬프트를 입력합니다. 예를 들면 다음과 같습니다.

    ```output
    What would you like to do?
    I'm planning an anniversary trip with my spouse, but they are currently using a wheelchair and accessibility is a must. What are some destinations that would be romantic for us?
    ```

1. 액세스 가능한 대상에 대한 권장 사항이 포함된 일부 출력을 받아야 합니다.

1. 작업 제안에 대한 프롬프트를 입력합니다. 예:

    ```output
    What would you like to do?
    What are some things to do in Barcelona?
    ```

1. 예를 들어, 다음과 유사한 바르셀로나에서 액세스 가능한 작업과 같이 이전 상황에 맞는 권장 사항을 받아야 합니다.

    ```output
    1. Visit the iconic Sagrada Família: This breathtaking basilica is an iconic symbol of Barcelona's architecture and is known for its unique design by Antoni Gaudí.

    2. Explore Park Güell: Another masterpiece by Gaudí, this park offers stunning panoramic views of the city, intricate mosaic work, and whimsical architectural elements.

    3. Visit the Picasso Museum: Explore the extensive collection of artworks by the iconic painter Pablo Picasso, showcasing his different periods and styles.
    ```

    > [!NOTE]
    > 코드가 예상한 결과를 생성하지 못하는 경우 **솔루션** 폴더에서 코드를 검토할 수 있습니다.

다양한 프롬프트와 상황 신호를 사용하여 애플리케이션을 계속 테스트할 수 있습니다. 잘하셨습니다. LLM에 상황 신호를 성공적으로 제공하고 사용자가 대화를 계속할 수 있도록 코드를 조정했습니다.

### 검토

이 랩에서는 LLM(대규모 언어 모델) 서비스에 대한 엔드포인트를 만들고, 의미 체계 커널 개체를 빌드하고, 의미 체계 커널 SDK를 사용하여 프롬프트를 실행하고, 의미 체계 커널 함수 및 플러그 인을 만들고, 의미 체계 커널 SDK의 자동 함수 호출 기능을 사용하여 사용자의 의도를 적절한 플러그 인으로 라우팅했습니다. 또한 대화 기록을 사용하여 LLM에 컨텍스트를 제공하고 사용자가 대화를 계속할 수 있도록 했습니다. 이 랩을 완료해 주셔서 감사합니다!

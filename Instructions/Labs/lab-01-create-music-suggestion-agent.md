---
lab:
  title: '랩: Azure OpenAI 및 의미 체계 커널 SDK를 사용하여 AI 에이전트 개발'
  module: 'Module 01: Build your kernel'
---

# 랩: AI 음악 추천 도우미 만들기
# 학생용 랩 매뉴얼

이 랩에서는 사용자의 음악 라이브러리를 관리하고 개인에게 맞는 노래 및 콘서트 추천을 제공할 수 있는 AI 도우미를 위한 코드를 만듭니다. 의미 체계 커널 SDK를 사용하여 AI 도우미를 빌드하고 LLM(대규모 언어 모델) 서비스에 연결합니다. 의미 체계 커널 SDK를 사용하면 LLM 서비스와 상호 작용하고 사용자 맞춤 추천을 제공할 수 있는 스마트 애플리케이션을 만들 수 있습니다.

## 랩 시나리오

여러분은 국제 오디오 스트리밍 서비스의 개발자입니다. 서비스를 AI와 통합하여 사용자 맞춤이 더 잘 구현된 환경을 제공하는 임무를 맡았습니다. AI는 사용자의 청취 기록과 선호도를 기반으로 음악 및 예정된 아티스트 콘서트를 추천할 수 있어야 합니다. 의미 체계 커널 SDK를 사용하여 LLM(대규모 언어 모델) 서비스와 상호 작용할 수 있는 AI 도우미를 빌드하기로 합니다.

## 목표

이 랩을 완료하면 다음을 달성할 수 있습니다.

* LLM(대규모 언어 모델) 서비스에 대한 엔드포인트 만들기
* 의미 체계 커널 개체 빌드
* 의미 체계 커널 SDK를 사용하여 프롬프트 실행
* 의미 체계 커널 함수 및 플러그 인 만들기
* 자동 함수 호출을 활성화하여 작업 자동화

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
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/01/Lab-01-Starter.zip`

1. 페이지의 오른쪽 위에 있는 `...`단추를 클릭하여 zip 파일을 다운로드하거나 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>S</kbd>를 누릅니다.

1. 데스크톱의 폴더와 같이 쉽게 찾고 기억할 수 있는 위치에 Zip 파일의 콘텐츠를 추출합니다.

1. Visual Studio Code를 열고 **파일** > **폴더 열기**를 선택합니다.

1. 추출한 **시작** 폴더로 이동하여 **폴더 선택**을 선택합니다.

1. 코드 편집기에서 **Program.cs** 파일을 엽니다.

> [!NOTE]
> 폴더에 있는 모든 파일을 신뢰하는지 묻는 메시지가 표시되면 **예, 작성자를 신뢰합니다**를 선택합니다. 

## 연습 1: 의미 체계 커널 SDK로 프롬프트 실행

이 연습에서는 LLM(대규모 언어 모델) 서비스에 대한 엔드포인트를 만듭니다. 의미 체계 커널 SDK는 이 엔드포인트를 사용하여 LLM에 연결하고 프롬프트를 실행합니다. 의미 체계 커널 SDK는 HuggingFace, OpenAI, Azure Open AI LLM을 지원합니다. 이 예제에서는 Azure Open AI를 사용합니다.

**예상 연습 완료 시간**: 10분

### 작업 1: Azure OpenAI 리소스 만들기

1. [https://portal.azure.com](https://portal.azure.com)으로 이동합니다.

1. 기본 설정을 사용하여 새 Azure OpenAI 리소스를 만듭니다.

    > [!NOTE]
    > Azure OpenAI 리소스가 이미 있는 경우 이 단계를 건너뛸 수 있습니다.

1. 리소스가 생성되면 **리소스로 이동**을 선택합니다.

1. **개요** 페이지에서 **Azure AI 파운드리 포털로 이동**을 선택합니다.

1. **새 배포 만들기**를 선택한 다음 **기본 모델에서**를 선택합니다.

1. 모델 목록에서 **gpt-35-turbo-16k**를 선택합니다.

1. **확인**을 선택합니다.

1. 배포의 이름을 입력하고 기본 옵션을 그대로 둡니다.

1. 배포가 완료되면 Azure Portal에서 Azure OpenAI 리소스로 다시 이동합니다.

1. **리소스 관리**에서 **키 및 엔드포인트**를 선택합니다.

    다음 작업에서 여기에 있는 데이터를 사용하여 커널을 빌드합니다. 키를 안전하게 비공개로 유지해야 합니다!

### 작업 2: 커널 빌드

이 연습에서는 첫 번째 의미 체계 커널 SDK 프로젝트를 빌드하는 방법을 알아봅니다. 새 프로젝트를 만들고, 의미 체계 커널 SDK NuGet 패키지를 추가하고, 의미 체계 커널 SDK에 대한 참조를 추가하는 방법을 알아봅니다. 그럼 시작하겠습니다.

1. Visual Studio Code 프로젝트로 돌아갑니다.

1. **appsettings.json** 파일을 열고 Azure OpenAI Services 모델 ID, 엔드포인트 및 API 키로 값을 업데이트합니다.

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. **터미널** > **새 터미널**을 선택하여 터미널을 엽니다.

1. 터미널에서 다음 명령을 실행하여 의미 체계 커널 SDK를 설치합니다.

    `dotnet add package Microsoft.SemanticKernel --version 1.30.0`

1. 1. 다음 `using` 지시문을 **Program.cs** 파일에 추가합니다.

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.ChatCompletion;
    using Microsoft.SemanticKernel.Connectors.OpenAI;
    ```

1. **Create a kernel builder with Azure OpenAI chat completion** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
    ```

1. **Build the kernel** 주석 아래에 이 코드를 추가하여 커널을 빌드합니다.

    ```c#
    // Build the kernel
    var kernel = builder.Build();
    ```

1. **Verify the endpoint and run a prompt** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Verify the endpoint and run a prompt
    var result = await kernel.InvokePromptAsync("Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. **시작** 폴더를 마우스 오른쪽 단추로 클릭하고 **통합 터미널에서 열기**를 선택합니다.

1. 터미널에 `dotnet run`을(를) 입력하여 코드를 실행합니다.

    세계에서 가장 유명한 뮤지션 상위 5명이 포함된 Azure Open AI 모델의 응답이 표시되는지 확인합니다.

    응답은 커널에 전달한 Azure Open AI 모델에서 제공됩니다. 의미 체계 커널 SDK는 LLM(대규모 언어 모델)에 연결하고 프롬프트를 실행할 수 있습니다. LLM으로부터 얼마나 빨리 응답을 받을 수 있었는지 확인합니다. 의미 체계 커널 SDK를 사용하면 스마트 애플리케이션을 쉽고 효율적으로 빌드할 수 있습니다.

응답을 확인한 뒤에는 확인 코드를 제거할 수 있습니다.

## 연습 2: 사용자 지정 음악 라이브러리 플러그 인 만들기

이 연습에서는 음악 라이브러리에 대한 사용자 지정 플러그 인을 만듭니다. 사용자의 최근 재생 목록에 노래를 추가하고, 최근 재생 노래 목록을 가져오고, 개인 맞춤 노래 추천을 제공할 수 있는 함수를 만듭니다. 또한 사용자의 위치와 최근 재생 노래를 기반으로 콘서트를 제안하는 함수를 만듭니다.

**예상 연습 완료 시간**: 15분

### 작업 1: 음악 라이브러리 플러그 인 만들기

이 작업에서는 사용자의 최근 재생 목록에 노래를 추가하고 최근 재생 노래 목록을 가져올 수 있는 플러그 인을 만듭니다. 간단히 하기 위해 최근에 재생한 노래는 텍스트 파일에 저장됩니다.

1. **Plugins** 폴더에서 **MusicLibraryPlugin.cs** 파일을 엽니다.

1. **Create a kernel function to get recently played songs** 주석 아래에 커널 함수 데코레이터를 추가합니다.


    ```c#
    // Create a kernel function to get recently played songs
    [KernelFunction("GetRecentPlays")]
    public static string GetRecentPlays()
    ```

    `KernelFunction` 데코레이터는 네이티브 함수를 선언합니다. AI가 함수를 올바르게 호출할 수 있도록 함수에 설명이 포함된 이름을 사용합니다. 사용자의 최근 재생 목록은 'RecentlyPlayed.txt'라는 텍스트 파일에 저장됩니다.

1. **Create a kernel function to add a song to the recently played list** 주석 아래에 커널 함수 데코레이터를 추가합니다.

    ```c#
    // Create a kernel function to add a song to the recently played list
    [KernelFunction("AddToRecentPlays")]
    public static string AddToRecentlyPlayed(string artist,  string song, string genre)
    ```

    이제 이 플러그인 클래스가 커널에 추가되면 함수를 식별하고 호출할 수 있습니다.

1. **Program.cs** 파일로 이동합니다.

1. **Import plugins to the kernel** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Import plugins to the kernel
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    ```

1. **Create prompt execution settings** 주석 아래에 다음 코드를 추가하여 함수를 자동으로 호출합니다.

    ```c#
    // Create prompt execution settings
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };
    ```

    이 설정을 사용하면 프롬프트에서 함수를 지정할 필요 없이 커널이 함수를 자동으로 호출할 수 있습니다.

1. **Get chat completion service** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Get chat completion service.
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
    ChatHistory chatHistory = [];
    ```

1. **Create a helper function to await and output the reply from the chat completion service** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Create a helper function to await and output the reply from the chat completion service
    async Task GetAssistantReply() {
        ChatMessageContent reply = await chatCompletionService.GetChatMessageContentAsync(
            chatHistory,
            kernel: kernel,
            executionSettings: openAIPromptExecutionSettings
        );
        chatHistory.AddAssistantMessage(reply.ToString());
        Console.WriteLine(reply.ToString());
    }
    ```


1. **Add system messages to the chat** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Add system messages to the chat
    chatHistory.AddSystemMessage("When a user has played a song, add it to their list of recent plays.");
    chatHistory.AddSystemMessage("The listener has just played the song Danse by Tiara. It falls under these genres: French pop, electropop, pop.");
    await GetAssistantReply();
    ```

1. 터미널에 `dotnet run`을 입력하여 코드를 실행합니다.

    추가한 시스템 메시지 프롬프트가 플러그인 함수를 호출하여 다음 출력이 표시되어야 합니다.

    ```output
    Added 'Danse' to recently played
    ```

    **Files/RecentlyPlayed.txt**를 열면 목록에 새 노래가 추가된 것을 볼 수 있습니다.

### 작업 2: 개인 맞춤 노래 추천 제공

이 작업에서는 최근에 재생한 노래를 기반으로 사용자 맞춤 노래 추천을 제공하는 프롬프트를 만듭니다. 프롬프트는 네이티브 함수를 결합하여 노래 추천을 생성합니다. 또한 프롬프트에서 함수를 만들어 다시 사용할 수 있도록 합니다.

1. **Program.cs** 파일의 **Create a song suggester function using a prompt** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Create a song suggester function using a prompt
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next",
        functionName: "SuggestSong",
        description: "Suggest a song to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);
    ```

    이 코드에서는 프롬프트에서 함수를 만들고 커널 플러그 인에 추가합니다.

1. **Invoke the song suggester function with a prompt from the user*** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Invoke the song suggester function with a prompt from the user
    chatHistory.AddUserMessage("What song should I play next?");
    await GetAssistantReply();
    ```

    이제 애플리케이션은 사용자의 요청에 따라 플러그인 함수를 자동으로 호출할 수 있습니다. 이 코드를 확장해 사용자의 정보를 기반으로 콘서트 추천을 제공하도록 만들어 보겠습니다.

### 작업 3: 맞춤형 콘서트 추천 제공

이 작업에서는 사용자의 최근 재생한 노래와 위치를 기반으로 콘서트를 추천하도록 LLM에 요청하는 플러그 인을 만듭니다.

1. **Program.cs** 파일의 **Import plugins to the kernel** 주석 아래에서 콘서트 플러그인을 가져옵니다.

    ```c#
    // Import plugins to the kernel 
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    ```

1. **Create a concert suggester function using a prompt** 주석 아래에 다음 코드를 추가합니다.

    ```c#
    // Create a concert suggester function using a prompt
    var concertSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"This is a list of the user's recently played songs:
        {{MusicLibraryPlugin.GetRecentPlays}}

        This is a list of upcoming concert details:
        {{MusicConcertsPlugin.GetConcerts}}

        Suggest an upcoming concert based on the user's recently played songs. 
        The user lives in {{$location}}, 
        please recommend a relevant concert that is close to their location.",
        functionName: "SuggestConcert",
        description: "Suggest a concert to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestConcertPlugin", [concertSuggesterFunction]);
    ```

    이 함수 프롬프트는 음악 라이브러리 및 예정된 콘서트 정보뿐만 아니라 사용자의 위치를 가져와 권장 사항을 제공합니다.

1. 다음 프롬프트를 추가하여 새 플러그 인 함수를 호출합니다.

    ```c#
    // Invoke the concert suggester function with a prompt from the user
    chatHistory.AddUserMessage("Can you recommend a concert for me? I live in Washington");
    await GetAssistantReply();
    ```

1. 터미널에서 `dotnet run`를 입력합니다.

    다음 응답과 유사한 출력이 표시됩니다.

    ```output
    I recommend you attend the concert of Lisa Taylor. She will be performing in Seattle, Washington, USA on 2/22/2024. Enjoy the show!
    ```
    
    LLM의 응답은 다를 수 있습니다. 프롬프트와 위치를 조정하여 어떤 다른 결과를 얻을 수 있는지 확인해 보세요.

이제 도우미가 사용자의 입력에 따라 자동으로 다양한 작업을 수행할 수 있습니다. 잘했습니다!

### 검토

이 랩에서는 사용자의 음악 라이브러리를 관리하고 개인에게 맞는 노래 및 콘서트 추천을 제공할 수 있는 AI 도우미를 만들었습니다. 의미 체계 커널 SDK를 사용하여 AI 도우미를 빌드하고 LLM(대규모 언어 모델) 서비스에 연결했습니다. 음악 라이브러리를 위한 사용자 지정 플러그인을 만들고 자동 함수 호출을 사용하도록 설정하여 도우미가 사용자의 입력에 동적으로 응답하도록 했습니다. 축하합니다. 랩을 완료했습니다.

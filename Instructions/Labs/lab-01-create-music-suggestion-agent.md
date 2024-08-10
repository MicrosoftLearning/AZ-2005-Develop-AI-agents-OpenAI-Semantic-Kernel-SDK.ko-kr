---
lab:
  title: '랩: Azure OpenAI 및 의미 체계 커널 SDK를 사용하여 AI 에이전트 개발'
  module: 'Module 01: Build your kernel'
---

# 랩: AI 음악 추천 에이전트 만들기
# 학생용 랩 매뉴얼

이 랩에서는 사용자의 음악 라이브러리를 관리하고 개인 맞춤 음악과 콘서트 추천을 제공할 수 있는 AI 에이전트를 위한 코드를 만듭니다. 의미 체계 커널 SDK를 사용하여 AI 에이전트를 빌드하고 LLM(대규모 언어 모델) 서비스에 연결합니다. 의미 체계 커널 SDK를 사용하면 LLM 서비스와 상호 작용하고 사용자 맞춤 추천을 제공할 수 있는 스마트 애플리케이션을 만들 수 있습니다.

## 랩 시나리오

여러분은 국제 오디오 스트리밍 서비스의 개발자입니다. 서비스를 AI와 통합하여 사용자 맞춤이 더 잘 구현된 환경을 제공하는 임무를 맡았습니다. AI는 사용자의 청취 기록과 선호도를 기반으로 음악 및 예정된 아티스트 콘서트를 추천할 수 있어야 합니다. 의미 체계 커널 SDK를 사용하여 LLM(대규모 언어 모델) 서비스와 상호 작용할 수 있는 AI 에이전트를 빌드하기로 합니다.

## 목표

이 랩을 완료하면 다음을 달성할 수 있습니다.

* LLM(대규모 언어 모델) 서비스에 대한 엔드포인트 만들기
* 의미 체계 커널 개체 빌드
* 의미 체계 커널 SDK를 사용하여 프롬프트 실행
* 의미 체계 커널 함수 및 플러그 인 만들기
* Handlebars 계획 도구를 사용하여 작업 자동화

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

1. **개요** 페이지에서 **Azure OpenAI Studio로 이동**을 선택합니다.

:::image type="content" source="../media/model-deployments.png" alt-text="Azure OpenAI 배포 페이지의 스크린샷.":::

1. **새 배포 만들기**를 선택한 다음, **모델 배포**를 선택합니다.

1. **모델 선택**에서 **gpt-35-turbo-16k**를 선택합니다.

    기본 모델 버전 사용

1. 배포 이름 입력

1. 배포가 완료되면 Azure OpenAI 리소스로 다시 이동합니다.

1. **리소스 관리**에서 **키 및 엔드포인트**를 선택합니다.

    다음 작업에서 여기에 있는 데이터를 사용하여 커널을 빌드합니다. 키를 안전하게 비공개로 유지해야 합니다!

### 작업 2: 커널 빌드

이 연습에서는 첫 번째 의미 체계 커널 SDK 프로젝트를 빌드하는 방법을 알아봅니다. 새 프로젝트를 만들고, 의미 체계 커널 SDK NuGet 패키지를 추가하고, 의미 체계 커널 SDK에 대한 참조를 추가하는 방법을 알아봅니다. 그럼 시작하겠습니다.

1. Visual Studio Code 프로젝트로 돌아갑니다.

1. **터미널** > **새 터미널**을 선택하여 터미널을 엽니다.

1. 터미널에서 다음 명령을 실행하여 의미 체계 커널 SDK를 설치합니다.

    `dotnet add package Microsoft.SemanticKernel --version 1.2.0`

1. 커널을 만들기 위해 다음 코드를 'Program.cs' 파일에 추가합니다.
    
    ```c#
    using Microsoft.SemanticKernel;

    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    ```

    자리 표시자를 Azure 리소스의 값으로 바꿔야 합니다.

1. 다음 코드를 입력하여 커널과 엔드포인트가 작동하는지 테스트합니다.

    ```c#
    var result = await kernel.InvokePromptAsync(
        "Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. `dotnet run`을(를) 입력하여 코드를 실행하고 세계에서 가장 유명한 뮤지션 상위 5명이 포함된 Azure Open AI 모델의 응답이 표시되는지 확인합니다.

    응답은 커널에 전달한 Azure Open AI 모델에서 제공됩니다. 의미 체계 커널 SDK는 LLM(대규모 언어 모델)에 연결하고 프롬프트를 실행할 수 있습니다. LLM으로부터 얼마나 빨리 응답을 받을 수 있었는지 확인합니다. 의미 체계 커널 SDK를 사용하면 스마트 애플리케이션을 쉽고 효율적으로 빌드할 수 있습니다.

## 연습 2: 사용자 지정 음악 라이브러리 플러그 인 만들기

이 연습에서는 음악 라이브러리에 대한 사용자 지정 플러그 인을 만듭니다. 사용자의 최근 재생 목록에 노래를 추가하고, 최근 재생 노래 목록을 가져오고, 개인 맞춤 노래 추천을 제공할 수 있는 함수를 만듭니다. 또한 사용자의 위치와 최근 재생 노래를 기반으로 콘서트를 제안하는 함수를 만듭니다.

**예상 연습 완료 시간**: 15분

### 작업 1: 음악 라이브러리 플러그 인 만들기

이 작업에서는 사용자의 최근 재생 목록에 노래를 추가하고 최근 재생 노래 목록을 가져올 수 있는 플러그 인을 만듭니다. 간단히 하기 위해 최근에 재생한 노래는 텍스트 파일에 저장됩니다.

1. 'Lab01-Project' 디렉터리에 새 폴더를 만들고 이름을 'Plugins'로 지정합니다.

1. 'Plugins' 폴더에 'MusicLibrary.cs'라는 새 파일을 만듭니다.

    먼저 사용자의 "최근 재생" 목록에 노래를 가져오고 추가하는 몇 가지 빠른 함수를 만듭니다.

1. 다음 코드를 입력합니다.

    ```c#
    using System.ComponentModel;
    using System.Text.Json;
    using System.Text.Json.Nodes;
    using Microsoft.SemanticKernel;

    public class MusicLibraryPlugin
    {
        [KernelFunction, 
        Description("Get a list of music recently played by the user")]
        public static string GetRecentPlays()
        {
            string content = File.ReadAllText($"Files/RecentlyPlayed.txt");
            return content;
        }
    }
    ```

    이 코드에서는 `KernelFunction` 데코레이터를 사용하여 네이티브 함수를 선언합니다. 또한 `Description` 데코레이터를 사용하여 함수의 기능에 대한 설명을 추가합니다. 사용자의 최근 재생 목록은 'RecentlyPlayed.txt'라는 텍스트 파일에 저장됩니다. 다음으로 목록에 노래를 추가하는 코드를 추가할 수 있습니다.

1. `MusicLibraryPlugin` 클래스에 다음 코드를 추가합니다.

    ```c#
    [KernelFunction, Description("Add a song to the recently played list")]
    public static string AddToRecentlyPlayed(
        [Description("The name of the artist")] string artist, 
        [Description("The title of the song")] string song, 
        [Description("The song genre")] string genre)
    {
        // Read the existing content from the file
        string filePath = "Files/RecentlyPlayed.txt";
        string jsonContent = File.ReadAllText(filePath);
        var recentlyPlayed = (JsonArray) JsonNode.Parse(jsonContent);

        var newSong = new JsonObject
        {
            ["title"] = song,
            ["artist"] = artist,
            ["genre"] = genre
        };

        recentlyPlayed.Insert(0, newSong);
        File.WriteAllText(filePath, JsonSerializer.Serialize(recentlyPlayed,
            new JsonSerializerOptions { WriteIndented = true }));

        return $"Added '{song}' to recently played";
    }
    ```

    이 코드에서는 아티스트, 노래, 장르를 문자열로 받아들이는 함수를 만듭니다. 함수의 `Description` 외에도 입력 변수에 대한 설명도 추가합니다. 'RecentlyPlayed.txt' 파일에는 사용자가 최근에 재생한 노래의 json 형식 목록이 포함되어 있습니다. 이 코드는 파일에서 기존 콘텐츠를 읽고 구문 분석한 후 새 노래를 목록에 추가합니다. 그런 다음 업데이트된 목록이 파일에 다시 기록됩니다.

1. Program.cs 파일을 다음 코드로 업데이트합니다.

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();

    var result = await kernel.InvokeAsync(
        "MusicLibraryPlugin", 
        "AddToRecentlyPlayed", 
        new() {
            ["artist"] = "Tiara", 
            ["song"] = "Danse", 
            ["genre"] = "French pop, electropop, pop"
        }
    );
    
    Console.WriteLine(result);
    ```

    이 코드에서는 `ImportPluginFromType`을 사용하여 `MusicLibraryPlugin`을 커널로 가져옵니다. 그런 다음 호출하려는 플러그 인 이름과 함수 이름으로 `InvokeAsync`를 호출합니다. 또한 아티스트, 노래, 장르를 인수로 전달합니다.

1. 터미널에 `dotnet run`을 입력하여 코드를 실행합니다.

    다음과 같은 출력이 표시됩니다.

    ```output
    Added 'Danse' to recently played
    ```

    'Files/RecentlyPlayed.txt,'를 열면 목록에 새 노래가 추가된 것을 볼 수 있습니다.

### 작업 2: 개인 맞춤 노래 추천 제공

이 작업에서는 최근에 재생한 노래를 기반으로 사용자 맞춤 노래 추천을 제공하는 프롬프트를 만듭니다. 프롬프트는 네이티브 함수를 결합하여 노래 추천을 생성합니다. 또한 프롬프트에서 함수를 만들어 다시 사용할 수 있도록 합니다.

1. `MusicLibraryPlugin.cs` 파일에 다음 함수를 추가합니다.

    ```c#
    [KernelFunction, Description("Get a list of music available to the user")]
    public static string GetMusicLibrary()
    {
        string dir = Directory.GetCurrentDirectory();
        string content = File.ReadAllText($"Files/MusicLibrary.txt");
        return content;
    }
    ```

    이 함수는 'MusicLibrary.txt'라는 파일에서 사용 가능한 음악 목록을 읽습니다. 파일에는 사용자가 사용할 수 있는 json 형식의 노래 목록이 포함되어 있습니다.

1. Program.cs 파일을 다음 코드로 업데이트합니다.

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    
    string prompt = @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next";

    var result = await kernel.InvokePromptAsync(prompt);
    Console.WriteLine(result);
    ```

이 코드에서는 네이티브 함수를 의미 체계 프롬프트와 결합합니다. 네이티브 함수는 LLM(대규모 언어 모델)이 자체적으로 액세스할 수 없는 사용자 데이터를 검색할 수 있으며, LLM은 텍스트 입력을 기반으로 권장하는 노래를 생성할 수 있습니다.

1. 코드를 테스트하려면 터미널에 `dotnet run`을 입력합니다.

    다음 출력과 유사한 응답이 표시됩니다.

    ```output 
    Based on the user's recently played music, a suggested song to play next could be "Sabry Aalil" by Yasemin since the user seems to enjoy pop and Egyptian pop music.
    ```

    >[!NOTE] 권장하는 노래는 여기에 표시된 노래와 다를 수 있습니다.

1. 프롬프트에서 함수를 만들도록 코드를 수정합니다.

    ```c#
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

    var result = await kernel.InvokeAsync(songSuggesterFunction);
    Console.WriteLine(result);
    ```

    이 코드에서는 사용자에게 노래를 제안하는 프롬프트에서 함수를 만듭니다. 그런 다음 커널 플러그 인에 추가합니다. 마지막으로 커널에 함수를 실행하도록 지시합니다.

### 작업 3: 맞춤형 콘서트 추천 제공

이 작업에서는 예정된 콘서트 세부 정보를 검색하는 플러그 인을 만듭니다. 또한 사용자가 최근에 재생한 노래와 사용자 위치에 따라 콘서트를 제안하도록 LLM에 요청하는 플러그 인을 만듭니다.

1. 'Plugins' 폴더에 'MusicConcertPlugin.cs'라는 새 파일을 만듭니다.

1. MusicConcertPlugin' 파일에 다음 코드를 추가합니다.

    ```c#
    using System.ComponentModel;
    using Microsoft.SemanticKernel;

    public class MusicConcertsPlugin
    {
        [KernelFunction, Description("Get a list of upcoming concerts")]
        public static string GetConcerts()
        {
            string content = File.ReadAllText($"Files/ConcertDates.txt");
            return content;
        }
    }
    ```

    이 코드에서는 'ConcertDates.txt'라는 파일에서 예정된 콘서트 목록을 읽는 함수를 만듭니다. 파일에는 예정된 콘서트의 json 형식 목록이 포함되어 있습니다. 다음으로 LLM에 콘서트 제안을 요청하는 프롬프트를 만들어야 합니다.

1. 'Prompts' 폴더에 'SuggestConcert'라는 새 폴더를 만듭니다.

1. 다음 콘텐츠로 'SuggestConcert' 폴더에 'config.json' 파일을 만듭니다.

    ```json
    {
        "schema": 1,
        "type": "completion",
        "description": "Suggest a concert to the user",
        "execution_settings": {
            "default": {
                "max_tokens": 4000,
                "temperature": 0
            }
        },
        "input_variables": [
            {
                "name": "location",
                "description": "The user's location",
                "required": true
            }
        ]
    }
    ```

1. 다음 콘텐츠로 'SuggestConcert' 폴더에 'skprompt.txt' 파일을 만듭니다.

    ```output
    This is a list of the user's recently played songs:
    {{MusicLibraryPlugin.GetRecentPlays}}

    This is a list of upcoming concert details:
    {{MusicConcertsPlugin.GetConcerts}}

    Suggest an upcoming concert based on the user's recently played songs. 
    The user lives in {{$location}}, 
    please recommend a relevant concert that is close to their location.
    ```

    이 프롬프트는 LLM이 사용자 입력을 필터링하고 텍스트에서 대상만 검색하는 데 도움이 됩니다. 다음으로 계획 도구를 호출하여 목표를 달성하기 위해 플러그 인을 결합하는 계획을 만듭니다.

1. 'Program.cs' 파일을 열고 다음 코드로 업데이트합니다.

    ```c#
    var kernel = builder.Build();    
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
    // code omitted for brevity
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);

    string location = "Redmond WA USA";
    var result = await kernel.InvokeAsync<string>(prompts["SuggestConcert"],
        new() {
            { "location", location }
        }
    );
    Console.WriteLine(result);
    ```

1. 터미널에서 `dotnet run`를 입력합니다.

    다음 응답과 유사한 출력이 표시됩니다.

    ```output
    Based on the user's recently played songs and their location in Redmond WA USA, a relevant concert recommendation would be the upcoming concert of Lisa Taylor in Seattle WA, USA on February 22, 2024. Lisa Taylor is an indie-folk artist, and her music genre aligns with the user's recently played songs, such as "Loanh Quanh" by Ly Hoa. Additionally, Seattle is close to Redmond, making it a convenient location for the user to attend the concert.
    ```

    프롬프트와 위치를 조정하여 어떤 다른 결과를 얻을 수 있는지 확인해 보세요.

## 연습 3: Handlebars 계획을 사용하여 제안 자동화

Handlebars 계획 도구는 작업을 수행하는 데 여러 단계가 필요할 때 유용합니다. 계획 도구는 AI를 사용하여 커널에 등록된 플러그 인을 선택하고 일련의 단계로 결합하여 목표를 달성합니다. 이 연습에서는 Handlebars 계획 도구를 사용하여 계획 템플릿을 생성하고 이를 사용하여 제안을 자동화합니다.

**예상 연습 완료 시간**: 10분

### 작업 1: 계획 템플릿 생성

이 작업에서는 Handlebars 계획 도구를 사용하여 계획 템플릿을 생성합니다. 계획 템플릿은 사용자의 입력에 따라 제안을 자동화하는 데 사용됩니다.

1. 터미널에 다음을 입력하여 Handlebars 계획 도구를 설치합니다.

    `dotnet add package Microsoft.SemanticKernel.Planners.Handlebars --version 1.2.0-preview`

    다음으로 SuggestConcert 프롬프트를 바꾸고 Handlebars 계획 도구를 사용하여 대신 작업을 수행하도록 합니다.

1. 'Program.cs' 파일에서 코드를 다음과 같이 업데이트합니다.

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.Planning.Handlebars;
    
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    kernel.ImportPluginFromPromptDirectory("Prompts");

    #pragma warning disable SKEXP0060
    var planner = new HandlebarsPlanner(new HandlebarsPlannerOptions() { AllowLoops = true });

    string location = "Redmond WA USA";
    string goal = @$"Based on the user's recently played music, suggest a 
        concert for the user living in ${location}";

    var plan = await planner.CreatePlanAsync(kernel, goal);
    var result = await plan.InvokeAsync(kernel);

    Console.WriteLine($"{result}");
    ```

    >[!NOTE] 핸들바 패키지는 현재 미리 보기 상태이므로 코드를 실행하려면 컴파일러 경고를 표시하지 않는 것이 필요할 수 있습니다.

1. 터미널에서 `dotnet run`를 입력합니다.

    다음 출력과 유사한 응답이 표시됩니다.

    ```output
    Based on the user's recently played songs, the artist "Mademoiselle" has an upcoming concert in Seattle WA, USA on February 22, 2024, which is close to Redmond WA. Therefore, the recommended concert for the user would be Mademoiselle's concert in Seattle.
    ```

    다음으로 코드를 변경하여 Handlebars 계획 템플릿을 출력하도록 합니다.

1. 'Program.cs' 파일에서 코드를 다음과 같이 업데이트합니다.

    ```c#
    var plan = await planner.CreatePlanAsync(kernel, goal);
    Console.WriteLine("Plan:");
    Console.WriteLine(plan);
    ```

    이제 생성된 계획을 볼 수 있습니다. 다음으로 노래 제안을 포함하거나 사용자의 최근 재생 목록에 노래를 추가하도록 계획을 수정합니다.

1. 다음 코드 조각을 사용하여 코드를 확장합니다.

    ```c#
    var plan = await planner.CreatePlanAsync(kernel, 
        @$"If add song:
        Add a song to the user's recently played list.
        
        If concert recommendation:
        Based on the user's recently played music, suggest a concert for 
        the user living in a given location.

        If song recommendation:
        Suggest a song from the music library to the user based on their 
        recently played songs.");

    Console.WriteLine("Plan:");
    Console.WriteLine(plan);
    ```

1. 만든 계획의 출력을 보려면 터미널에 `dotnet run`을 입력합니다.

    다음 출력과 유사한 템플릿이 표시됩니다.

    ```output
    Plan:
    {{!-- Step 1: Identify Key Values --}}
    {{set "location" location}}
    {{set "addSong" addSong}}
    {{set "concertRecommendation" concertRecommendation}}
    {{set "songRecommendation" concertRecommendation}}

    {{!-- Step 2: Use the Right Helpers --}}
    {{#if addSong}}
        {{set "song" song}}
        {{set "artist" artist}}
        {{set "genre" genre}}
        {{set "songAdded" (MusicLibraryPlugin-AddToRecentlyPlayed artist=artist song=song genre=genre)}}  
        {{json songAdded}}
    {{/if}}

    {{#if concertRecommendation}}
        {{set "concertSuggested" (Prompts-SuggestConcert location=location recentlyPlayedSongs=recentlyPlayedSongs musicLibrary=musicLibrary)}}
        {{json concertSuggested}}
    {{/if}}

    {{#if songRecommendation}}
        {{set "songSuggested" (SuggestSongPlugin-SuggestSong recentlyPlayedSongs=recentlyPlayedSongs musicLibrary=musicLibrary)}}
        {{json songSuggested}}
    {{/if}}

    {{!-- Step 3: Output the Result --}}
    {{json "Goal achieved"}}
    ```

     `{{#if ...}}` 구문을 확인합니다. 이 구문은 C#의 기존 `if`-`else` 블록과 유사하게 Handlebars 계획 도구가 사용할 수 있는 조건문 역할을 합니다. `if` 문은 `{{/if}}`로 닫혀야 합니다.

    그다음, 생성된 이 템플릿을 사용하여 사용자 고유의 Handlebars 계획을 만듭니다. 

1. 'Files' 디렉토리에서 다음 텍스트를 사용하여 'HandlebarsTemplate.txt'라는 새 파일을 만듭니다.

    ```output
    {{set "addSong" addSong}}
    {{set "concertRecommendation" concertRecommendation}}
    {{set "songRecommendation" songRecommendation}}

    {{#if addSong}}
        {{set "song" song}}
        {{set "artist" artist}}
        {{set "genre" genre}}
        {{set addedSong (MusicLibraryPlugin-AddToRecentlyPlayed artist song genre)}}  
        Output The following content, do not make any modifications:
        {{json addedSong}}     
    {{/if}}

    {{#if concertRecommendation}}
        {{set "location" location}}
        {{set "concert" (Prompts-SuggestConcert location)}}
        Output The following content, do not make any modifications:
        {{json concert}}
    {{/if}}

    {{#if songRecommendation}}
        {{set "recentlyPlayedSongs" (MusicLibraryPlugin-GetRecentPlays)}}
        {{set "musicLibrary" (MusicLibraryPlugin-GetMusicLibrary)}}
        {{set "song" (SuggestSongPlugin-SuggestSong recentlyPlayedSongs musicLibrary)}}
        Output The following content, do not make any modifications:
        {{json song}}
    {{/if}}
    ```

    이 템플릿에서는 텍스트를 생성하지 않도록 LLM에 명령을 추가하여 출력이 플러그 인에 의해 엄격하게 관리되도록 합니다. 이제 템플릿을 사용해 보겠습니다.

### 작업 2: Handlebars 계획 도구를 사용하여 제안 자동화

이 작업에서는 Handlebars 계획 템플릿에서 함수를 만들고 이를 사용하여 사용자의 입력에 따라 제안을 자동화합니다.

1. 기존 코드를 수정하여 Handlebars 계획을 제거합니다.

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.PromptTemplates.Handlebars;

    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    kernel.ImportPluginFromPromptDirectory("Prompts");
    
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"Based on the user's recently played music:
        {{$recentlyPlayedSongs}}
        recommend a song to the user from the music library:
        {{$musicLibrary}}",
        functionName: "SuggestSong",
        description: "Suggest a song to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);
    ```

1. 템플릿 파일을 읽는 코드를 추가하면 함수가 만들어집니다.

    ```c#
    string template = File.ReadAllText($"Files/HandlebarsTemplate.txt");

    var handlebarsPromptFunction = kernel.CreateFunctionFromPrompt(
        new() {
            Template = template,
            TemplateFormat = "handlebars"
        }, new HandlebarsPromptTemplateFactory()
    );
    ```

    이 코드에서는 `Template` 개체를 `TemplateFormat`과 함께 커널 메서드 `CreateFunctionFromPrompt`에 전달합니다. `CreateFunctionFromPrompt`는 또한 지정된 템플릿을 구문 분석하는 방법을 커널에 알려 주는 `IPromptTemplateFactory` 형식을 허용합니다. Handlebars 템플릿을 사용하고 있으므로 `HandlebarsPromptTemplateFactory` 형식을 사용합니다.

    다음으로 몇 가지 인수를 사용하여 함수를 실행하고 결과를 확인해 보겠습니다.

1. `Program.cs` 파일에 다음 코드를 추가합니다.

    ```c#
    string location = "Redmond WA USA";
    var templateResult = await kernel.InvokeAsync(handlebarsPromptFunction,
        new() {
            { "location", location },
            { "concertRecommendation", true },
            { "songRecommendation", false },
            { "addSong", false },
            { "artist", "" },
            { "song", "" },
            { "genre", "" }
        });

    Console.WriteLine(templateResult);
    ```

1. 계획 도구 템플릿의 출력을 보려면 터미널에 `dotnet run`을 입력합니다.

    다음 출력과 유사한 응답이 표시됩니다.

    ```output
    Based on the user's recently played songs, Ly Hoa seems to be a relevant artist. The closest concert to Redmond WA, USA would be in Portland OR, USA on April 16th, 2024.  
    ```

    프롬프트는 최근에 재생한 음악 목록과 사용자 위치를 기반으로 사용자에게 콘서트를 제안할 수 있었습니다. 또한 다른 변수를 true로 설정하고 어떤 일이 일어나는지 확인해 볼 수 있습니다.

이제 코드는 사용자의 입력에 따라 다른 작업을 수행할 수 있습니다. 잘했습니다!

### 검토

이 랩에서는 사용자의 음악 라이브러리를 관리하고 개인 맞춤 노래 및 콘서트 추천을 제공할 수 있는 AI 에이전트를 만들었습니다. 의미 체계 커널 SDK를 사용하여 AI 에이전트를 빌드하고 LLM(대규모 언어 모델) 서비스에 연결했습니다. 음악 라이브러리에 대한 사용자 지정 플러그 인을 만들고, Handlebars 계획 도구를 사용하여 제안을 자동화하고, Handlebars 계획 템플릿에서 함수를 만들어 사용자의 입력에 따라 제안을 자동화했습니다. 축하합니다. 랩을 완료했습니다.

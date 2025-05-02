---
lab:
  title: 의미 체계 커널 SDK를 사용하여 Devops 도우미 만들기
---

# 의미 체계 커널 SDK를 사용하여 Devops 도우미 만들기

이 랩에서는 TODO인 AI 도우미에 대한 코드를 만듭니다. 의미 체계 커널 SDK를 사용하여 AI 도우미를 빌드하고 LLM(대규모 언어 모델) 서비스에 연결합니다. 의미 체계 커널 SDK를 사용하면 LLM 서비스와 상호 작용하고 사용자 맞춤 추천을 제공할 수 있는 스마트 애플리케이션을 만들 수 있습니다.

## 채팅 완료 모델 배포

1. [https://portal.azure.com](https://portal.azure.com)으로 이동합니다.

1. 기본 설정을 사용하여 새 Azure OpenAI 리소스를 만듭니다.

1. 리소스가 생성되면 **리소스로 이동**을 선택합니다.

1. **개요** 페이지에서 **Azure Foundry 포털로 이동**을 선택합니다.

1. **새 배포 만들기**를 선택한 다음 **기본 모델에서** 선택합니다.

1. 모델 목록에서 **gpt-4o**를 검색한 후 선택하고 확인합니다.

1. 배포의 이름을 입력하고 기본 옵션을 그대로 둡니다.

1. 배포가 완료되면 Azure Portal에서 Azure OpenAI 리소스로 다시 이동합니다.

1. **리소스 관리**에서 **키 및 엔드포인트**를 선택합니다.

    다음 작업에서 여기에 있는 데이터를 사용하여 커널을 빌드합니다. 키를 안전하게 비공개로 유지해야 합니다!

## 애플리케이션 구성 준비

1. 새 브라우저 탭을 엽니다(Azure AI 파운드리 포털을 기존 탭에서 열어 두기). 그런 다음 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.

    Azure Portal 홈페이지를 보려면 환영 알림을 닫습니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 구독에 저장소가 없는 ***PowerShell*** 환경을 선택합니다.

    Cloud Shell은 Azure Portal 하단의 창에서 명령줄 인터페이스를 제공합니다. 보다 쉽게 작업할 수 있도록 이 창의 크기를 조정하거나 최대화할 수 있습니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. Cloud Shell 창에서 다음 명령을 입력하여 이 연습의 코드 파일이 포함된 GitHub 리포지토리를 복제합니다(명령을 입력하거나 클립보드에 복사한 다음 명령줄을 마우스 오른쪽 단추로 클릭하여 일반 텍스트로 붙여넣습니다).

    ```
    rm -r semantic-kernel -f
    git clone https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK semantic-kernel
    ```

    > **팁**: CloudShell에 명령을 붙여넣으면 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 후 채팅 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.

    > **참고**: 선택한 프로그래밍 언어에 대한 단계를 따릅니다.

    **Python**
    ```
    cd semantic-kernel/Allfiles/Labs/Devops/python
    ```

    **C#**
    ```
    cd semantic-kernel/Allfiles/Labs/Devops/c-sharp
    ```

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    **Python**
    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    **C#**
    ```
    dotnet add package Microsoft.Extensions.Configuration
    dotnet add package Microsoft.Extensions.Configuration.Json
    dotnet add package Microsoft.SemanticKernel
    dotnet add package Microsoft.SemanticKernel.PromptTemplates.Handlebars
    ```

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    **Python**
    ```
    code .env
    ```

    **C#**
    ```
    code appsettings.json
    ```

    코드 편집기에서 파일이 열립니다.

1. Azure OpenAI Services 모델 ID, 엔드포인트 및 API 키로 값을 업데이트합니다.

    **Python**
    ```python
    MODEL_DEPLOYMENT=""
    BASE_URL=""
    API_KEY="
    ```

    **C#**
    ```json
    {
        "modelName": "",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. 값을 업데이트한 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음, Cloud Shell 명령줄을 열어 둔 채 **Ctrl+Q** 명령을 사용하여 코드 편집기를 닫습니다.

## 의미 체계 커널 플러그 인 만들기

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    **Python**
    ```
    code devops.py
    ```

    **C#**
    ```
    code Program.cs
    ```

1. **Create a kernel builder with Azure OpenAI chat completion** 주석 아래에 다음 코드를 추가합니다.

    **Python**
    ```python
    # Create a kernel builder with Azure OpenAI chat completion
    kernel = Kernel()
    chat_completion = AzureChatCompletion(
        deployment_name=deployment_name,
        api_key=api_key,
        base_url=base_url,
    )
    kernel.add_service(chat_completion)
    ```
    **C#**
     ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
    var kernel = builder.Build();
    ```

1. 파일 아래쪽에서 **Create a kernel function to build the stage environment** 주석을 찾고 다음 코드를 추가하여 스테이징 환경을 빌드할 모의 플러그인 기능을 만듭니다.

    **Python**
    ```python
    # Create a kernel function to build the stage environment
    @kernel_function(name="BuildStageEnvironment")
    def build_stage_environment(self):
        return "Stage build completed."
    ```

    **C#**
    ```c#
    // Create a kernel function to build the stage environment
    [KernelFunction("BuildStageEnvironment")]
    public string BuildStageEnvironment() 
    {
        return "Stage build completed.";
    }
    ```

    `KernelFunction` 데코레이터는 네이티브 함수를 선언합니다. AI가 함수를 올바르게 호출할 수 있도록 함수에 설명이 포함된 이름을 사용합니다. 

1. **Import plugins to the kernel** 주석으로 이동하여 다음 코드를 추가합니다.

    **Python**
    ```python
    # Import plugins to the kernel
    kernel.add_plugin(DevopsPlugin(), plugin_name="DevopsPlugin")
    ```

    **C#**
    ```c#
    // Import plugins to the kernel
    kernel.ImportPluginFromType<DevopsPlugin>();
    ```


1. **Create prompt execution settings** 주석 아래에 다음 코드를 추가하여 함수를 자동으로 호출합니다.

    **Python**
    ```python
    # Create prompt execution settings
    execution_settings = AzureChatPromptExecutionSettings()
    execution_settings.function_choice_behavior = FunctionChoiceBehavior.Auto()
    ```

    **C#**
    ```c#
    // Create prompt execution settings
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };
    ```

    이 설정을 사용하면 프롬프트에서 함수를 지정할 필요 없이 커널이 함수를 자동으로 호출할 수 있습니다.

1. **Create chat history** 주석 아래에 다음 코드를 추가합니다.

    **Python**
    ```python
    # Create chat history
    chat_history = ChatHistory()
    ```

    **C#**
    ```c#
    // Create chat history
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
    ChatHistory chatHistory = [];
    ```

1. **User interaction logic** 주석 뒤에 있는 코드 블록의 주석 처리를 제거합니다.

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다.

## devops 도우미 코드를 실행합니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 Azure에 로그인합니다.

    ```
    az login
    ```

    **<font color="red">Cloud Shell 세션이 이미 인증되었더라도 Azure에 로그인해야 합니다.</font>**

    > **참고**: 대부분의 시나리오에서는 *az login*을 사용하는 것만으로도 충분합니다. 그러나 여러 테넌트에 구독이 있는 경우 *--tenant* 매개 변수를 사용하여 테넌트 지정해야 할 수 있습니다. 자세한 내용은 [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)를 참조하세요.

1. 메시지가 표시되면 지침에 따라 새 탭에서 로그인 페이지를 열고 제공된 인증 코드와 Azure 자격 증명을 입력합니다. 그런 다음 명령줄에서 로그인 프로세스를 완료하고 메시지가 표시되면 Azure AI 파운드리 허브가 포함된 구독을 선택합니다.

1. 로그인한 후 다음 명령을 입력하여 애플리케이션을 실행합니다.


    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. 메시지가 표시되면 다음 프롬프트를 입력합니다.`Please build the stage environment`

1. 다음 출력과 유사한 응답이 표시됩니다.

    ```output
    Assistant: The stage environment has been successfully built.
    ```

1. 그리고 다음 프롬프트를 입력합니다.`Please deploy the stage environment`

1. 다음 출력과 유사한 응답이 표시됩니다.

    ```output
    Assistant: The staging site has been deployed successfully.
    ```

1. <kbd>ENTER</kbd> 키를 눌러 프로그램을 종료합니다.

## 프롬프트에서 커널 함수 만들기

1. 주석 아래에 다음 코드를 추가합니다.`Create a kernel function to deploy the staging environment`

     **Python**
    ```python
    # Create a kernel function to deploy the staging environment
    deploy_stage_function = KernelFunctionFromPrompt(
        prompt="""This is the most recent build log:
        {{DevopsPlugin.ReadLogFile}}

        If there are errors, do not deploy the stage environment. Otherwise, invoke the stage deployment function""",
        function_name="DeployStageEnvironment",
        description="Deploy the staging environment"
    )

    kernel.add_function(plugin_name="DeployStageEnvironment", function=deploy_stage_function)
    ```

    **C#**
    ```c#
    // Create a kernel function to deploy the staging environment
    var deployStageFunction = kernel.CreateFunctionFromPrompt(
    promptTemplate: @"This is the most recent build log:
    {{DevopsPlugin.ReadLogFile}}

    If there are errors, do not deploy the stage environment. Otherwise, invoke the stage deployment function",
    functionName: "DeployStageEnvironment",
    description: "Deploy the staging environment"
    );

    kernel.Plugins.AddFromFunctions("DeployStageEnvironment", [deployStageFunction]);
    ```

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 애플리케이션을 실행합니다.

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. 메시지가 표시되면 다음 프롬프트를 입력합니다.`Please deploy the stage environment`

1. 다음 출력과 유사한 응답이 표시됩니다.

    ```output
    Assistant: The stage environment cannot be deployed because the earlier stage build failed due to unit test errors. Deploying a faulty build to stage may cause eventual issues and compromise the environment.
    ```

    LLM의 응답은 다양할 수 있지만 여전히 스테이지 사이트를 배포할 수 없습니다.

## Handlebars 프롬프트 만들기

1. **핸들바 프롬프트 만들기** 주석 아래에 다음 코드를 추가합니다.

    **Python**
    ```python
    # Create a handlebars prompt
    hb_prompt = """<message role="system">Instructions: Before creating a new branch for a user, request the new branch name and base branch name/message>
        <message role="user">Can you create a new branch?</message>
        <message role="assistant">Sure, what would you like to name your branch? And which base branch would you like to use?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">"""
    ```

    **C#**
    ```c#
    // Create a handlebars prompt
    string hbprompt = """
        <message role="system">Instructions: Before creating a new branch for a user, request the new branch name and base branch name/message>
        <message role="user">Can you create a new branch?</message>
        <message role="assistant">Sure, what would you like to name your branch? And which base branch would you like to use?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">
        """;
    ```

    이 코드에서는 핸들바 템플릿 형식을 사용하여 퓨샷 프롬프트를 만듭니다. 프롬프트는 새 분기를 만들기 전에 모델이 사용자로부터 더 많은 정보를 검색하도록 안내합니다.

1. **핸들바 형식을 사용하여 프롬프트 템플릿 구성 만들기** 주석 아래에 다음 코드를 추가합니다.

    **Python**
    ```python
    # Create the prompt template config using handlebars format
    hb_template = HandlebarsPromptTemplate(
        prompt_template_config=PromptTemplateConfig(
            template=hb_prompt, 
            template_format="handlebars",
            name="CreateBranch", 
            description="Creates a new branch for the user",
            input_variables=[
                InputVariable(name="input", description="The user input", is_required=True)
            ]
        ),
        allow_dangerously_set_content=True,
    )
    ```

    **C#**
    ```c#
    // Create the prompt template config using handlebars format
    var templateFactory = new HandlebarsPromptTemplateFactory();
    var promptTemplateConfig = new PromptTemplateConfig()
    {
        Template = hbprompt,
        TemplateFormat = "handlebars",
        Name = "CreateBranch",
    };
    ```

    이 코드에서는 프롬프트에서 핸들바 템플릿 구성을 만듭니다. 이를 사용하여 플러그 인 함수를 만들 수 있습니다.

1. **프롬프트에서 플러그인 함수 만들기** 주석 아래에 다음 코드를 추가합니다. 

    **Python**
    ```python
    # Create a plugin function from the prompt
    prompt_function = KernelFunctionFromPrompt(
        function_name="CreateBranch",
        description="Creates a branch for the user",
        template_format="handlebars",
        prompt_template=hb_template,
    )
    kernel.add_function(plugin_name="BranchPlugin", function=prompt_function)
    ```

    **C#**
    ```c#
    // Create a plugin function from the prompt
    var promptFunction = kernel.CreateFunctionFromPrompt(promptTemplateConfig, templateFactory);
    var branchPlugin = kernel.CreatePluginFromFunctions("BranchPlugin", [promptFunction]);
    kernel.Plugins.Add(branchPlugin);
    ```

    그런 다음 프롬프트에 대한 플러그 인 함수를 만들어 커널에 추가합니다. 이제 함수를 호출할 준비가 되었습니다.

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 애플리케이션을 실행합니다.

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. 메시지가 표시되면 다음 텍스트를 입력합니다.`Please create a new branch`

1. 다음 출력과 유사한 응답이 표시됩니다.

    ```output
    Assistant: Could you please provide the following details?

    1. The name of the new branch.
    2. The base branch from which the new branch should be created.
    ```

1. 다음 텍스트를 입력합니다.`feature-login main`

1. 다음 출력과 유사한 응답이 표시됩니다.

    ```output
    Assistant: The new branch `feature-login` has been successfully created from `main`.
    ```

## 작업에 대한 사용자 동의 필요

1. 파일 아래쪽 근처에서 **Create a function filter** 주석을 찾아 다음 코드를 추가합니다.

    **Python**
    ```python
    async def permission_filter(context: FunctionInvocationContext, next: Callable[[FunctionInvocationContext], Awaitable[None]]) -> None:
        await next(context)
        result = context.result
        
        # Check the plugin and function names
    ```

    **C#**
    ```c#
    // Create a function filter
    class PermissionFilter : IFunctionInvocationFilter
    {
        public async Task OnFunctionInvocationAsync(FunctionInvocationContext context, Func<FunctionInvocationContext, Task> next)
        {
            // Check the plugin and function names
            
            await next(context);
        }
    }
    ```

1. `DeployToProd` 함수가 호출되는 시기를 감지하려면 **Check the plugin and function names** 주석 아래에 다음 코드를 추가합니다.

     **Python**
    ```python
    # Check the plugin and function names
    if context.function.plugin_name == "DevopsPlugin" and context.function.name == "DeployToProd":
        # Request user approval
        
        # Proceed if approved
    ```

    **C#**
    ```c#
    // Check the plugin and function names
    if ((context.Function.PluginName == "DevopsPlugin" && context.Function.Name == "DeployToProd"))
    {
        // Request user approval

        // Proceed if approved
    }
    ```

    이 코드는 `FunctionInvocationContext` 개체를 사용하여 어떤 플러그 인과 함수가 호출되었는지 확인합니다.

1. 항공편을 예약하기 위한 사용자의 허가를 요청하려면 다음 논리를 추가합니다.

     **Python**
    ```python
    # Request user approval
    print("System Message: The assistant requires approval to complete this operation. Do you approve (Y/N)")
    should_proceed = input("User: ").strip()

    # Proceed if approved
    if should_proceed.upper() != "Y":
        context.result = FunctionResult(
            function=result.function,
            value="The operation was not approved by the user",
        )
    ```

    **C#**
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

1. **Add filters to the kernel** 주석으로 이동하여 다음 코드를 추가합니다.

    **Python**
    ```python
    # Add filters to the kernel
    kernel.add_filter('function_invocation', permission_filter)
    ```

    **C#**
    ```c#
    // Add filters to the kernel
    kernel.FunctionInvocationFilters.Add(new PermissionFilter());
    ```

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 애플리케이션을 실행합니다.

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. 프로덕션에 빌드를 배포하라는 프롬프트를 입력합니다. 다음과 유사한 응답이 표시됩니다.

    ```output
    User: Please deploy the build to prod
    System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)
    User: N
    Assistant: I'm sorry, but I am unable to proceed with the deployment.
    ```

### 검토

이 랩에서는 LLM(대규모 언어 모델) 서비스에 대한 엔드포인트를 만들고, 의미 체계 커널 개체를 빌드하고, 의미 체계 커널 SDK를 사용하여 프롬프트를 실행했습니다. 또한, 모델을 안내하기 위해 플러그인을 만들고 시스템 메시지를 활용했습니다. 축하합니다. 랩을 완료했습니다.
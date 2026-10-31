---
title: "Анализ покрытия тестами кода для C#/.NET"
num: 4
summary: "Coverlet, dotnet-coverage и HTML-отчёт о покрытии"
---

# Анализ покрытия тестами кода для C#/.NET

Эта статья описывает, как подключить и использовать анализ покрытия кода тестами для проекта на C# 14 / .NET 10 — например, для бэкенда из [референсного проекта](https://github.com/is-aidd-26/plain-old-messenger).

## Подключение

Конечным результатом настройки будет скрипт, который запускает все тесты проекта и генерирует HTML-отчёт о покрытии кода тестами по пути `tests/coverage-report/index.html`.

### Шаг 1. Установка инструментов

Для обработки результатов потребуются dotnet tools, которые можно установить так:

```powershell
dotnet tool install --global dotnet-coverage
dotnet tool install --global dotnet-reportgenerator-globaltool
```

### Шаг 2. Создание файла RunSettings

В файле RunSettings мы разместим настройки для coverlet, позволяющие отфильтровать лишнее из отчёта о покрытии кода тестами.

Создайте в проекте файл `tests/tests.runsettings` со следующим содержимым:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<RunSettings>
    <DataCollectionRunSettings>
        <DataCollectors>
            <DataCollector friendlyName="XPlat Code Coverage">
                <Configuration>
                    <Format>cobertura</Format>
                    <ExcludeByFile>**/tests/**/*.cs;**/tests/*.cs</ExcludeByFile>
                    <ExcludeByAttribute>GeneratedCodeAttribute</ExcludeByAttribute>
                    <SkipAutoProps>true</SkipAutoProps>
                </Configuration>
            </DataCollector>
        </DataCollectors>
    </DataCollectionRunSettings>
</RunSettings>
```

### Шаг 3. Добавление зависимости от пакета coverlet.collector

Эта зависимость автоматически добавляется при создании теста из шаблона xunit.

Однако вы можете проверить, что во всех файлах `*.csproj` проектов тестов — например, в `tests/ChatApi.UnitTests/ChatApi.UnitTests.csproj` — есть зависимость `coverlet.collector`:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <IsPackable>false</IsPackable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="coverlet.collector" Version="6.0.4" />
    <!-- ... -->
  </ItemGroup>

</Project>
```

### Шаг 4. Доработка .gitignore

Добавьте в файл `.gitignore` в корне проекта игнорирование результатов сбора покрытия кода тестами:

```
# Данные о покрытии кода тестами
tests/coverage/
tests/coverage-report/
```

### Шаг 5. Добавление скриптов запуска

#### Bash-скрипт для GNU/Linux

Если используете GNU/Linux, то добавьте скрипт `scripts/run-tests-with-coverage`:

```bash
#!/usr/bin/env bash
#
# Запускает тесты со сбором данных о покрытии кода тестами.
# Обрабатывает эти данные, генерируя отчёт о покрытии тестами.
#
# Установка инструментов для обработки данных о покрытии кода тестами:
#   dotnet tool install --global dotnet-coverage
#   dotnet tool install --global dotnet-reportgenerator-globaltool
#
# По завершению скрипта создаётся HTML-отчёт: tests/coverage-report/index.html

set -o errexit

PROJECT_DIR=$(dirname "$(dirname "$(readlink -f "$0")")")

echo_call() {
    echo "$@"
    "$@"
}

pushd "$PROJECT_DIR" >/dev/null
echo_call rm -rf tests/coverage/
echo_call rm -rf tests/coverage-report/
echo_call dotnet test --settings tests/tests.runsettings --collect "XPlat Code Coverage" --results-directory=tests/coverage/
echo_call dotnet-coverage merge tests/coverage/*/*.xml --output tests/coverage/merged.cobertura.xml --output-format cobertura
echo_call reportgenerator -reports:tests/coverage/merged.cobertura.xml -targetdir:tests/coverage-report/ -reporttypes:Html
popd >/dev/null
```

Не забудьте сделать скрипт исполняемым: `chmod +x scripts/run-tests-with-coverage`

#### PowerShell-скрипт для Windows

Если используете Windows, то добавьте скрипт `scripts/run-tests-with-coverage.ps1`:

```powershell
#!/usr/bin/env powershell
#
# Запускает тесты со сбором данных о покрытии кода тестами.
# Обрабатывает эти данные, генерируя отчёт о покрытии тестами.
#
# Установка инструментов для обработки данных о покрытии кода тестами:
#   dotnet tool install --global dotnet-coverage
#   dotnet tool install --global dotnet-reportgenerator-globaltool
#
# По завершению скрипта создаётся HTML-отчёт: tests/coverage-report/index.html

Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

$ProjectDir = Split-Path -Path $PSScriptRoot -Parent

# Печатает команду, затем запускает её и проверяет код возврата.
function EchoAndCall {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory = $true, Position = 0)]
        [string]$Command,

        [Parameter(ValueFromRemainingArguments = $true)]
        [string[]]$Arguments
    )

    # Сохраняем в переменную экранированные аргументы
    $formattedArgs = ($Arguments | ForEach-Object {
        if ($_ -match '\s|;|=|"') {
            "`"$_`""  # Экранируем аргументы с пробелами/спецсимволами
        } else {
            $_
        }
    }) -join ' '

    Write-Host "$Command $formattedArgs"

    # Сбрасываем последний код возврата
    $LASTEXITCODE = 0

    # Запускаем внешнюю команду
    & $Command $Arguments

    # Проверяем код возврата внешней команды
    if ($LASTEXITCODE -ne 0) {

        throw "Command '$Command $formattedArgs' failed with exit code $LASTEXITCODE."
    }
}

pushd $ProjectDir
Remove-Item -Path tests/coverage/ -Recurse -ErrorAction Ignore
Remove-Item -Path tests/coverage-report/ -Recurse -ErrorAction Ignore
EchoAndCall -- dotnet test --settings tests/tests.runsettings --collect "XPlat Code Coverage" --results-directory=tests/coverage/
EchoAndCall -- dotnet-coverage merge tests/coverage/*/*.xml --output tests/coverage/merged.cobertura.xml --output-format cobertura
EchoAndCall -- reportgenerator "-reports:tests/coverage/merged.cobertura.xml" "-targetdir:tests/coverage-report/" "-reporttypes:Html"
popd
```

## Использование

Для запуска тестов с генерацией отчёта о покрытии запустите скрипт в терминале:

```bash
scripts/run-tests-with-coverage
```

После этого скопируйте в VSCode (или ином редакторе) полный путь к `tests/coverage-report/index.html` и откройте его в браузере.

На главной странице обратите внимание на раздел "Coverage" и кнопки "Collapse all", "Expand all":

![Главная страница HTML-отчёта о покрытии](/assets/images/code-coverage-report.png)

На этом скриншоте (взят из другого учебного проекта) видно, что покрытие тестами сборки "Lexer" составляет 74.2%. Можно кликнуть на названии сборки, чтобы раскрыть её, а затем перейти к классу "Lexer.Lexer". На открывшейся странице можно увидеть непокрытые тестами методы, например, `TryParseEscapeSequence`:

![Непокрытый метод в отчёте](/assets/images/code-coverage-uncovered-method.png)

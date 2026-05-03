# Laravel Exception Analyzer
A Laravel package that helps you understand your errors - not just log them.

It can be installed directly into a Laravel project and will automatically collect, group, and analyze exceptions to give you a clear overview of what’s actually going wrong.

⚠️ Proof of concept built for a real-world use case

## 📌 The Problem
This project was built for a company where all errors were sent directly to the communication platform Slack.

At peak times, they received 300+ error messages per hour.

The result:
- no real overview
- repeated errors looked like new ones
- important issues were buried in noise

👉 It became almost impossible to understand what was actually wrong.

## 💡 The Solution
This package replaces noisy error streams with structured insight.

It:
- captures exceptions automatically
- stores them in a database
- groups similar errors together
- tracks how often they occur
- detects when issues are resolved

👉 Instead of hundreds of messages, you get a few clear problems to focus on.

## 🔄 How It Works
1) The package hooks into Laravel’s exception handling
2) All exceptions are stored automatically
3) Background jobs analyze and group errors
4) Recurring issues are tracked over time
5) Issues are marked as resolved when they stop occurring

This allows developers to:
- identify recurring issues
- understand error patterns over time
- prioritize fixes based on impact
- track when problems are actually resolved

## 🔄 System Overview

The analyzer is part of a larger exception-handling flow:
1) An exception occurs in a Laravel application
2) Laravel’s built-in ExceptionHandler is triggered
3) A custom reportable callback intercepts the exception
4) The exception is stored as raw data in the database
5) A scheduled process analyzes and structures the data
6) Exceptions are grouped into recurring issues
7) Resolved issues are automatically detected and closed

This design ensures that no exception is lost, while analysis happens asynchronously for stability.

## Tech stack
- Docker so we had the same environment in development.
- Mistral AI (locally on each pc)
- Laravel (PHP web application framework)
- Frontend: Blade, HTML, CSS 

## 🧠 Key Concepts
### Raw vs Structured Exceptions
- Raw exceptions are stored immediately when they occur
- Structured exceptions are created later through AI-based classification

This separation ensures that failures in the analysis layer never affect data collection.

### Exception Grouping (CFL)
Exceptions are grouped using a CFL (Carrier-File-Line) identifier:
- identifies the technical origin of an error
- enables grouping of similar exceptions
- provides insight into recurring issues

This makes it possible to detect patterns instead of treating errors individually.

### Repetitive Exceptions
Recurring issues are stored as RepetitiveExceptions, which:
- represent a known problem
- are continuously updated if the issue persists
- are automatically marked as resolved when no longer occurring

This ensures that only active problems remain visible.

### AI-Based Classification
Exceptions are analyzed using an AI component that:
- classifies severity
- determines whether the error is internal or external
- extracts meaningful metadata

The AI returns structured JSON, making results directly usable in the system.

# Sekvens (Danish / dansk) – hele kæden som trin-for-trin
1) En exception opstår i Laravel-applikationen.


2) Laravel’s exception-system (via ExceptionHandler/Exceptions facade) kalder det reportable callback, som facaden (Facades\LaravelExceptionAnalyzer) har registreret i handles().


3) Facaden tjekker i config('laravel-exception-analyzer'), om analysen er slået til (isEnabled).
- Hvis den er slået fra → der sker ikke mere.
- Hvis den er slået til → facaden resolver LaravelExceptionAnalyzer-servicen fra containeren og kalder $analyzer->report($exception).


4) LaravelExceptionAnalyzer er hovedservicen. Den har en injiceret ReportClient og delegere arbejdet videre til den via report($exception).


5) ReportClient har en injiceret AiClient.
   I ReportClient::report():
- kalder den $this->aiClient->classify($exception).


6) AiClient:
- læser AI-config (enabled, api_key, endpoint, timeout)
- hvis AI er deaktiveret eller config mangler → returnerer null
- ellers saniterer den exceptionen via ExceptionSanitizer::sanitize()
- sender den saniterede payload til AI-service via Http::post()
- hvis AI svarer succesfuldt med JSON, mapper den svaret til et AiClassificationResult-objekt


7) ReportClient får AiClassificationResult tilbage (eller null ved fejl).
- Hvis resultatet findes, kan det logges eller gemmes i databasen.
- I vores nuværende kode logges resultatet (eller er klar til at blive gemt, når DB-delen er klar).


8) På den måde bliver hver exception automatisk klassificeret mht.:
- category
- source
- severity
- status_message 

# Installation

You can install the package via composer:

```bash
composer require 150657373-nikolajve/laravel-exception-analyzer
```

You can publish and run the migrations with:

```bash
php artisan vendor:publish --tag="laravel-exception-analyzer-migrations"
php artisan migrate
```

You can publish the config file with:

```bash
php artisan vendor:publish --tag="laravel-exception-analyzer-config"
```

This is the contents of the published config file:

```php
return [
];
```

Optionally, you can publish the views using

```bash
php artisan vendor:publish --tag="laravel-exception-analyzer-views"
```

## Usage

```php
$laravelExceptionAnalyzer = new NikolajVE\LaravelExceptionAnalyzer();
echo $laravelExceptionAnalyzer->echoPhrase('Hello, NikolajVE!');
```

## Testing

```bash
composer test
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security Vulnerabilities

Please review [our security policy](../../security/policy) on how to report security vulnerabilities.

## Credits

- [NikolajVE](https://github.com/150657373+NikolajVE)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

🚀 Flutter Flavors: Criando Múltiplas Versões do Seu App (Para Iniciantes!)

Imagine que você está construindo um aplicativo, como um jogo. Você quer ter:

    Uma versão de teste para você e sua equipe experimentarem novas funcionalidades.
    Uma versão de "demonstração" para mostrar para investidores ou testadores beta.
    A versão final que vai para a loja de aplicativos.

Cada uma dessas versões pode ter pequenas diferenças, como:

    O nome do aplicativo (ex: "Meu Jogo - Teste", "Meu Jogo - Demo", "Meu Jogo").
    O ícone do aplicativo (para diferenciar visualmente).
    O endereço do servidor com o qual o aplicativo se comunica (um servidor de teste, um servidor de produção, etc.).
    Recursos extras ativados ou desativados (como ferramentas de debug).

Flutter Flavors é a ferramenta que permite criar essas diferentes versões do seu app de forma organizada e fácil!
🤔 O Que São Flavors?

"Flavor" em inglês significa "sabor". No contexto do Flutter, um flavor é como uma "receita" diferente para construir seu aplicativo. Cada "sabor" (flavor) tem suas próprias configurações, mas todos compartilham o mesmo código base.
🛠️ Como Usar Flavors: Passo a Passo

Vamos criar um exemplo simples com três flavors: development (desenvolvimento), staging (teste/demonstração) e production (produção).
1. Configurando o Android (arquivo android/app/build.gradle)

Abra o arquivo android/app/build.gradle que fica dentro da pasta android, depois dentro da pasta app do seu projeto Flutter. É como se fosse o "documento de identidade" do seu app para o Android.

Dentro desse arquivo, procure a seção android { ... }. Dentro dela, adicione o seguinte código (se já existir algo parecido, adapte):

android {
    // ... outras configurações ...

    flavorDimensions "environment" // 👈 Adicione esta linha

    productFlavors { // 👈 Adicione este bloco
        development {
            dimension "environment"
            applicationIdSuffix ".dev"  // Ex: com.meuapp.dev
            resValue "string", "app_name", "Meu App Dev" // Nome do app
        }
        staging {
            dimension "environment"
            applicationIdSuffix ".stg"  // Ex: com.meuapp.stg
            resValue "string", "app_name", "Meu App Staging"
        }
        production {
            dimension "environment"
            // Sem applicationIdSuffix, usa o applicationId padrão
            resValue "string", "app_name", "Meu App"
        }
    }
}

O que fizemos aqui?

    flavorDimensions "environment": Criamos um "grupo" de flavors chamado "environment". É como uma categoria para organizar os sabores.
    productFlavors { ... }: Aqui definimos cada flavor:
        development: Para desenvolvimento.
            applicationIdSuffix ".dev": Adiciona ".dev" ao final do ID do aplicativo (ex: com.suaempresa.seuapp vira com.suaempresa.seuapp.dev). Isso é importante para poder instalar várias versões do app no mesmo celular.
            resValue "string", "app_name", "Meu App Dev": Define o nome do aplicativo como "Meu App Dev".
        staging: Para testes e demonstração. Configurações similares ao development.
        production: A versão final. Não usamos applicationIdSuffix aqui, então ele usa o ID padrão do seu app.

2. Criando a Lógica no Flutter (arquivo lib/main.dart)

Agora, vamos para o código Flutter, no arquivo lib/main.dart. Este é o "coração" do seu aplicativo.

import 'package:flutter/material.dart';

// 1. Criamos um "tipo" para os ambientes (como um rótulo)
enum Environment { development, staging, production }

// 2. Criamos uma "caixa" para guardar as configurações
class Config {
  final String apiUrl; // Endereço do servidor
  final String appName; // Nome do app
  final bool enableLogging; // Mostrar ou não informações de debug
  final Environment environment; // Qual o ambiente atual

  // Construtor privado (não se preocupe com isso agora)
  Config._internal({
    required this.apiUrl,
    required this.appName,
    required this.enableLogging,
    required this.environment,
  });

  // Variáveis para o Singleton (explicado abaixo)
  static Config? _instance;
  static late Environment _environment;

  // Método para definir o ambiente (ex: development, staging...)
  static void setEnvironment(Environment env) {
    _environment = env;
  }

  // "Fábrica" de configurações (Singleton)
  static Config get instance {
    _instance ??= _getConfig(); // Cria a instância se ainda não existir
    return _instance!;
  }

  // Função que escolhe a configuração certa
  static Config _getConfig() {
    switch (_environment) {
      case Environment.development:
        return Config._internal(
          apiUrl: 'https://dev-api.example.com', // Servidor de desenvolvimento
          appName: 'Meu App Dev',
          enableLogging: true, // Mostrar logs
          environment: Environment.development,
        );
      case Environment.staging:
        return Config._internal(
          apiUrl: 'https://staging-api.example.com', // Servidor de staging
          appName: 'Meu App Staging',
          enableLogging: true,
          environment: Environment.staging,
        );
      case Environment.production:
        return Config._internal(
          apiUrl: 'https://api.example.com', // Servidor de produção
          appName: 'Meu App',
          enableLogging: false, // Não mostrar logs
          environment: Environment.production,
        );
    }
  }
}

// Função principal do app
void main() {
  // Pega o flavor do ambiente (ex: "development", "staging"...)
  const flavor = String.fromEnvironment('FLAVOR', defaultValue: 'development');

  // Define o ambiente baseado no flavor
  switch (flavor) {
    case 'development':
      Config.setEnvironment(Environment.development);
      break;
    case 'staging':
      Config.setEnvironment(Environment.staging);
      break;
    case 'production':
      Config.setEnvironment(Environment.production);
      break;
    default:
      Config.setEnvironment(Environment.development); // Valor padrão
  }

  runApp(const MyApp()); // Inicia o app
}

// O resto do seu app (MyApp, MyHomePage, etc.)
class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: Config.instance.appName, // Usa o nome do app da configuração
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(Config.instance.appName), // Usa o nome do app
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Ambiente: ${Config.instance.environment}'), // Mostra o ambiente
            const SizedBox(height: 10),
            Text('API URL: ${Config.instance.apiUrl}'), // Mostra a URL da API
            const SizedBox(height: 10),
            Text('Logging: ${Config.instance.enableLogging}'), // Mostra se o logging está ligado
          ],
        ),
      ),
    );
  }
}

O que fizemos aqui?

    enum Environment: Criamos um tipo especial chamado Environment que só pode ter três valores: development, staging ou production. É como uma lista de opções.
    class Config: Criamos uma classe para guardar as configurações de cada ambiente.
        apiUrl, appName, enableLogging, environment: São as informações que variam entre os ambientes.
        Singleton (_instance, get instance): Isso garante que só exista uma "caixa" de configurações no app inteiro. É como ter um único controle remoto para a TV.
        _getConfig(): Esta função escolhe a configuração correta baseado no ambiente atual.
    void main():
        String.fromEnvironment('FLAVOR', ...): Aqui, o Flutter "pergunta" qual flavor deve ser usado. Essa informação vem de fora (veremos como passar essa informação no próximo passo).
        Config.setEnvironment(...): Define o ambiente baseado no flavor recebido.
        runApp(const MyApp()): Inicia o aplicativo.
    MyApp e MyHomePage: O restante do código do seu aplicativo, que agora usa as configurações do Config.instance.

3. Executando o App com Diferentes Flavors

Agora, a parte mágica! Para rodar o app com um flavor específico, você precisa usar um comando um pouco mais longo no terminal (ou configurar seu editor de código):

# Para o flavor development:
flutter run --flavor development --dart-define=FLAVOR=development

# Para o flavor staging:
flutter run --flavor staging --dart-define=FLAVOR=staging

# Para o flavor production:
flutter run --flavor production --dart-define=FLAVOR=production

O que significam esses comandos?

    flutter run: O comando normal para executar o app.
    --flavor <nome_do_flavor>: Diz ao Flutter qual flavor usar (ex: development, staging, production). Ele vai procurar essa configuração lá no build.gradle.
    --dart-define=FLAVOR=<nome_do_flavor>: Passa a informação do flavor para o código Dart (para a função main()).

4. Configurando o VS Code (Opcional)

Se você usa o VS Code, pode criar um arquivo chamado .vscode/launch.json para facilitar a execução dos flavors:

{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Development",
      "request": "launch",
      "type": "dart",
      "args": [
        "--flavor",
        "development",
        "--dart-define=FLAVOR=development"
      ]
    },
    {
      "name": "Staging",
      "request": "launch",
      "type": "dart",
      "args": [
        "--flavor",
        "staging",
        "--dart-define=FLAVOR=staging"
      ]
    },
    {
      "name": "Production",
      "request": "launch",
      "type": "dart",
      "args": [
        "--flavor",
        "production",
        "--dart-define=FLAVOR=production"
      ]
    }
  ]
}

Agora, você pode selecionar o flavor diretamente na aba de Debug do VS Code!
🖼️ Ilustrações (Exemplos)

Exemplo 1: Diferentes Ícones

Você pode ter ícones diferentes para cada flavor. No Android, você colocaria os ícones em pastas diferentes dentro de android/app/src/:

    android/app/src/development/res/mipmap-... (ícones para development)
    android/app/src/staging/res/mipmap-... (ícones para staging)
    android/app/src/main/res/mipmap-... (ícones para production - main é o padrão)

Exemplo 2: Diferentes Telas Iniciais (Splash Screens)

Da mesma forma, você pode ter telas iniciais diferentes para cada flavor.
📝 Resumo

    Flavors permitem criar diferentes versões do seu app (desenvolvimento, teste, produção).
    Configuramos os flavors no android/app/build.gradle (para Android).
    Usamos uma classe Config no Flutter para gerenciar as configurações de cada flavor.
    Executamos o app com flutter run --flavor ... --dart-define=FLAVOR=....
    Podemos usar o VS Code para facilitar a execução dos flavors.

Com Flutter Flavors, você pode organizar seu projeto de forma profissional e ter controle total sobre as diferentes versões do seu aplicativo!


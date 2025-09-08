CourtAILoginApp
iOS-приложение для аутентификации пользователей через Google и Apple с использованием Firebase Authentication. После успешного входа токен отправляется на бэкенд (https://api.court360.ai/rpc/client), сохраняется в Keychain, и пользователь переходит на экран приветствия с вкладками "Домой" и "Профиль". Приложение имеет современный UI с градиентами, тенями и закруглёнными углами, адаптировано под тёмную тему.
Важно: Бэкенд https://api.court360.ai/rpc/client в данный момент возвращает HTTP 521 (Cloudflare: Web Server Is Down), что указывает на недоступность сервера. Для полного тестирования требуется рабочий endpoint или доступ к бэкенду. Свяжитесь с администратором API для решения.
Инструкция по запуску
Требования

Xcode: 15.0+ (рекомендуется последняя версия).
iOS: 13.0+ (рекомендуется тестировать на iOS 17+).
Устройство: iPhone/iPad (симулятор на M1/M2 Mac может вызывать проблемы с Google Sign-In).
Apple ID: нужен платный аккаунт разработчика.
Зависимости: Firebase/Auth и GoogleSignIn-iOS (устанавливаются через Swift Package Manager).
Интернет: Для аутентификации и связи с бэкендом.

Установка

Клонируйте репозиторий:
git clone <repository-url>
cd CourtAILoginApp


Добавьте зависимости через Swift Package Manager:

В Xcode: File > Add Packages:
Firebase: https://github.com/firebase/firebase-ios-sdk (выберите Firebase/Auth).
GoogleSignIn: https://github.com/google/GoogleSignIn-iOS (версия 7.1.0+).


Нажмите Add Package.


Настройте подпись в Xcode:

Откройте проект в Xcode (CourtAILoginApp.xcodeproj).
Перейдите в Project > Target > Signing & Capabilities:
Укажите Team: Ваш Apple ID (Personal Team для бесплатного аккаунта).
Укажите Bundle Identifier: com.courtai.login (или ваш, совпадающий с Firebase).
Добавьте capability Sign In with Apple.
Нажмите Fix Issue, если Xcode предложит.




Настройте Info.plist:

Добавьте GIDClientID:<key>GIDClientID</key>
<string>YOUR_CLIENT_ID</string>

Замените YOUR_CLIENT_ID на значение CLIENT_ID из GoogleService-Info.plist (см. Firebase Setup).
Добавьте URL Types:
Identifier: google.
URL Schemes: REVERSED_CLIENT_ID из GoogleService-Info.plist (например, com.googleusercontent.apps.123456789-abc123).




Добавьте GoogleService-Info.plist:

Перетащите файл GoogleService-Info.plist (полученный из Firebase Console) в корень проекта в Xcode (включите "Copy items if needed").
Убедитесь, что файл содержит CLIENT_ID и REVERSED_CLIENT_ID.


Соберите и запустите:

Подключите iPhone/iPad (iOS 17+) или выберите симулятор.
Очистите проект: Product > Clean Build Folder (Shift+Cmd+K).
Нажмите Run (▶️).
На устройстве: Доверяйте сертификату в Settings > General > VPN & Device Management > Trust "Apple Development: your@email.com".


Тестирование:

На экране входа нажмите Войти через Google или Sign in with Apple.
Если бэкенд (https://api.court360.ai/rpc/client) недоступен (HTTP 521), вы увидите алерт с ошибкой "Ошибка ответа сервера".
Проверьте логи в Xcode Console (View > Debug Area > Activate Console) для отладки:
"Using clientID: ..." — подтверждает корректную настройку Google Sign-In.
"HTTP Status: 521" — указывает на проблему с сервером.


Для успешного тестирования требуется рабочий бэкенд. Свяжитесь с администратором API.



Firebase Setup
Для работы аутентификации через Google и Apple необходимо настроить Firebase проект.
Шаги

Создайте проект в Firebase:

Перейдите в Firebase Console.
Нажмите Add Project, следуйте инструкциям.
Добавьте iOS-приложение:
Bundle ID: com.courtai.login (должен совпадать с Xcode).
Скачайте GoogleService-Info.plist и добавьте в проект (см. выше).




Включите провайдеры аутентификации:

В Firebase Console > Authentication > Sign-in method:
Google: Включите, укажите email поддержки проекта.
Apple: Включите, убедитесь, что capability Sign In with Apple добавлена в Xcode.


Сохраните изменения. Если GoogleService-Info.plist обновился, перескачайте его.


Проверьте Google Cloud Console (для Google Sign-In):

В Google Cloud Console:
Найдите OAuth 2.0 Client ID для iOS (заканчивается на .apps.googleusercontent.com).
Убедитесь, что Bundle ID совпадает с com.courtai.login.
Если лимит клиентов (~36) превышен, удалите старые неиспользуемые клиенты.


Проверьте, что CLIENT_ID и REVERSED_CLIENT_ID в GoogleService-Info.plist совпадают с Google Cloud.


Проверка конфигурации:

Убедитесь, что FirebaseApp.configure() вызывается в App.swift перед любыми операциями с Firebase.
Проверьте логи в Xcode Console, чтобы убедиться, что CLIENT_ID загружается корректно.



Архитектура приложения
Приложение построено на SwiftUI с архитектурой MVVM (Model-View-ViewModel), что обеспечивает чистый, модульный и масштабируемый код.

Model:
AuthResponse, User, JSONRPCResponse: Структуры для декодирования ответа бэкенда в формате JSON-RPC.
KeychainHelper: Утилита для безопасного хранения accessToken в Keychain с использованием kSecClassGenericPassword.


View:
ContentView: Корневой вью с NavigationStack, переключающий между LoginView и WelcomeView на основе состояния AuthState.isLoggedIn.
LoginView: Экран входа с градиентным фоном (синий/фиолетовый), кнопками Google и Apple, тенями, закруглёнными углами и алертами для ошибок.
WelcomeView: Экран приветствия с градиентом (зелёный/синий), отображающий имя пользователя и переход на MainView.
MainView: Главный экран с TabView (вкладки "Домой" и "Профиль"), содержащий плейсхолдер-контент (список и профиль).


ViewModel:
AuthViewModel: Управляет логикой аутентификации (Firebase Auth, Google/Apple Sign-In, отправка fbIdToken на бэкенд). Использует async/await для современных сетевых вызовов и Result для обработки ошибок.
AuthState: Хранит состояние авторизации (isLoggedIn, userName) с использованием @Published для реактивного обновления UI.


Особенности:
Асинхронные сетевые запросы через URLSession с JSON-RPC для бэкенда.
Безопасное хранение токена в Keychain.
Обработка ошибок (например, HTTP 521) с выводом в алерт.
Современный UI с адаптацией под тёмную тему.



Комментарии:

MVVM идеально подходит для SwiftUI благодаря @ObservableObject и реактивности.
KeychainHelper обеспечивает безопасность, но не проверяет срок действия токена (см. продакшен-улучшения).
UI минималистичен и готов к расширению по макетам Figma (если предоставлены).
Проблема с бэкенд-сервером (HTTP 521) не влияет на корректность кода, но требует решения со стороны API.

Адаптация под тёмную тему
Приложение полностью адаптировано под тёмную тему благодаря использованию SwiftUI:

Цвета: Используются системные цвета (.white, .black, .blue), которые автоматически адаптируются к светлой и тёмной темам.
Градиенты в LoginView (синий/фиолетовый) и WelcomeView (зелёный/синий) выглядят гармонично в обоих режимах за счёт .opacity(0.6–0.8).
Тени (shadow с .opacity(0.3)) остаются читаемыми.


Тестирование:
На устройстве: Переключите в Settings > Display & Brightness > Dark.
В Xcode: Включите тёмный режим в симуляторе (Settings > Developer > Dark Appearance).


Рекомендации для улучшения:
Добавить кастомные цвета в Assets.xcassets с вариантами для светлой/тёмной темы:extension Color {
    static let primaryGradientStart = Color("PrimaryGradientStart") // Определите в Assets
    static let primaryGradientEnd = Color("PrimaryGradientEnd")
}


Использовать @Environment(\.colorScheme) для специфичных стилей:@Environment(\.colorScheme) var colorScheme
var body: some View {
    LinearGradient(
        gradient: Gradient(colors: colorScheme == .dark ? [.gray.opacity(0.8), .black.opacity(0.6)] : [.blue.opacity(0.8), .purple.opacity(0.6)]),
        startPoint: .top,
        endPoint: .bottom
    )
}


Добавить поддержку Dynamic Type для доступности текста:Text("CourtAI")
    .font(.system(.largeTitle, design: .rounded).weight(.bold))
    .dynamicTypeSize(.large)





Что можно улучшить в продакшен-версии
Для подготовки приложения к продакшену рекомендуется внедрить следующие улучшения:

Обработка сетевых ошибок:

Добавить retry-механизм для запросов к бэкенду (например, 3 попытки с экспоненциальной задержкой):private func sendTokenToBackend(idToken: String, retries: Int = 3) async throws -> Result<AuthResponse, Error> {
    for attempt in 1...retries {
        do {
            let result = try await sendTokenToBackend(idToken: idToken)
            return result
        } catch {
            if attempt == retries { throw error }
            try await Task.sleep(nanoseconds: UInt64(attempt) * 1_000_000_000)
        }
    }
    return .failure(NSError(domain: "Retry", code: -1, userInfo: [NSLocalizedDescriptionKey: "Все попытки исчерпаны"]))
}


Логировать ошибки в Firebase Crashlytics для мониторинга:
Добавить FirebaseCrashlytics через Swift Package Manager.
Логировать ошибки:import FirebaseCrashlytics
Crashlytics.crashlytics().record(error: error)






Валидация токена:

Проверять срок действия accessToken при запуске приложения:class AuthState: ObservableObject {
    init() {
        if let token = KeychainHelper.loadToken(), !token.isEmpty {
            Task {
                let isValid = await AuthViewModel().validateToken(token)
                await MainActor.run { isLoggedIn = isValid }
            }
        }
    }
}


Реализовать метод validateToken в AuthViewModel (зависит от API).


UI/UX:

Добавить анимации переходов между экранами для плавного UX:NavigationStack {
    if authState.isLoggedIn {
        WelcomeView()
            .transition(.slide)
    } else {
        LoginView()
            .transition(.slide)
    }
}
.animation(.easeInOut, value: authState.isLoggedIn)


Реализовать реальный контент в MainView (вкладки "Домой" и "Профиль") на основе макетов Figma.
Добавить кнопку "Выйти" в WelcomeView или ProfileTabView:Button("Выйти") {
    try? Auth.auth().signOut()
    KeychainHelper.saveToken("")
    authState.isLoggedIn = false
    authState.userName = "Пользователь"
}
.font(.headline)
.padding()
.background(Color.red)
.foregroundColor(.white)
.cornerRadius(10)




Сетевые улучшения:

Использовать библиотеку Alamofire для упрощения HTTP-запросов и логирования:
Добавить Alamofire через Swift Package Manager.
Заменить URLSession в sendTokenToBackend на:import Alamofire
let response = await AF.request(url, method: .post, parameters: body, encoding: JSONEncoding.default).serializingDecodable(JSONRPCResponse.self)




Кэшировать idToken локально и обновлять при истечении срока действия (используя Auth.auth().currentUser?.getIDTokenForcingRefresh(true)).


Тестирование:

Добавить unit-тесты для AuthViewModel (например, для методов randomNonceString, sha256):func testRandomNonceString() {
    let nonce = AuthViewModel().randomNonceString(length: 32)
    XCTAssertEqual(nonce.count, 32)
    XCTAssertTrue(nonce.allSatisfy { "abcdefABCDEF1234567890".contains($0) })
}


Добавить UI-тесты для проверки переходов между LoginView, WelcomeView и MainView.
Настроить CI/CD (например, GitHub Actions) для автоматической сборки и тестирования.


Безопасность:

Проверять подпись JWT-токена (fbIdToken, accessToken) перед отправкой/использованием.
Использовать App Attest (iOS 14+) для защиты от подделки устройства.
Добавить защиту от MITM-атак через SSL Pinning в URLSession или Alamofire.


Локализация:

Добавить поддержку нескольких языков для текстов (например, "Войти через Google", "Ошибка"):Text("login.google.button".localized)


Создать Localizable.strings для переводов.


Мониторинг производительности:

Интегрировать Firebase Performance Monitoring для отслеживания времени запросов.
Добавить метрики загрузки UI (например, время отображения LoginView).



Известные проблемы

Бэкенд недоступен: Сервер https://api.court360.ai/rpc/client возвращает HTTP 521 (Cloudflare: Web Server Is Down). Для тестирования требуется рабочий endpoint. Свяжитесь с администратором API для предоставления альтернативного URL или статуса сервера.
Google Sign-In: Если возникает ошибка "No active configuration. Make sure GIDClientID is set in Info.plist", проверьте:
Наличие GIDClientID и URL Types в Info.plist.
Корректность GoogleService-Info.plist (содержит CLIENT_ID и REVERSED_CLIENT_ID).
Наличие AppDelegate в App.swift для обработки URL-обратных вызовов.


RBS-ошибки: Логи RBSAssertionErrorDomain и RBSServiceErrorDomain могут появляться при использовании бесплатного Apple ID. Решение:
Убедитесь, что Team и Bundle ID настроены в Signing & Capabilities.
Пересоберите приложение после истечения 7-дневного профиля (для бесплатного аккаунта).



Лицензия
MIT License (или замените на вашу, если требуется).

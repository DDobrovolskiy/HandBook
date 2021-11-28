Link: https://qastack.ru/programming/41480102/how-spring-security-filter-chain-works

Ключевые фильтры в цепочке есть (в заказе)

* SecurityContextPersistenceFilter (восстанавливает аутентификацию из JSESSIONID)
* UsernamePasswordAuthenticationFilter (выполняет аутентификацию)
* ExceptionTranslationFilter (перехватывать исключения безопасности из FilterSecurityInterceptor)
* FilterSecurityInterceptor (может выдавать исключения аутентификации и авторизации)
Изучив текущую документацию стабильного выпуска 4.2.1 , раздел 13.3 Упорядочение фильтров, вы можете увидеть организацию фильтров всей цепочки фильтров:

13.3 Порядок фильтров

Порядок, в котором фильтры определяются в цепочке, очень важен. Независимо от того, какие фильтры вы на самом деле используете, порядок должен быть следующим:

1. ChannelProcessingFilter , потому что может потребоваться перенаправление на другой протокол

2. SecurityContextPersistenceFilter , поэтому SecurityContext может быть установлен в SecurityContextHolder в начале веб-запроса, а любые изменения в SecurityContext могут быть скопированы в HttpSession, когда веб-запрос заканчивается (готов для использования со следующим веб-запросом)

3. ConcurrentSessionFilter , потому что он использует функциональность SecurityContextHolder и должен обновить SessionRegistry, чтобы отразить текущие запросы от участника

4. Механизмы обработки аутентификации - UsernamePasswordAuthenticationFilter , CasAuthenticationFilter , BasicAuthenticationFilter и т. Д., Так что SecurityContextHolder можно изменить, чтобы он содержал действительный токен запроса аутентификации

5. SecurityContextHolderAwareRequestFilter , если вы используете его для установки Spring Security известно HttpServletRequestWrapper в ваш контейнер сервлетов

6. JaasApiIntegrationFilter , если JaasAuthenticationToken находится в SecurityContextHolder это будет обрабатывать FilterChain как субъект в JaasAuthenticationToken

7. RememberMeAuthenticationFilter , так что если более ранний механизм обработки аутентификации не обновил SecurityContextHolder, а запрос представляет файл cookie, который позволяет запускать сервисы запомнить меня, туда будет помещен подходящий запомненный объект аутентификации.

8. AnonymousAuthenticationFilter , так что если ранее механизм обработки аутентификации не обновлял SecurityContextHolder, туда будет помещен анонимный объект аутентификации

9. ExceptionTranslationFilter , для перехвата любых исключений Spring Security, чтобы можно было либо вернуть ответ об ошибке HTTP, либо запустить соответствующий AuthenticationEntryPoint.

10. FilterSecurityInterceptor , для защиты веб-URI и создания исключений, когда доступ запрещен
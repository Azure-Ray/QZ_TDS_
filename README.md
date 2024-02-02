@Component
@Order(-2) // Make sure it has higher priority than WebFlux's ErrorWebExceptionHandler
public class CustomGlobalExceptionHandler implements WebExceptionHandler {

    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
        if (ex instanceof BadJwtException) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        // 对于其他异常，保持默认处理
        return Mono.error(ex);
    }
}

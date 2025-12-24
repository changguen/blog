- Filter
    - Filter는 Servlet 실행 전후로 처리를 해주는 로직을 담는 곳이다.
    - Filter는 정상 처리가 된 경우에는 파라미터로 받은 chain의 doFilter를 통해 다음 필터를 실행시켜줘야 한다.
    
    ```java
    default void init(FilterConfig filterConfig) throws ServletException { }
    
    void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;
    
    default void destroy() { }
    ```
    
- FilterChain
    - doFilter를 통해서 Filter들을 실행하고, 모든 Filter를 통과한 경우에는 Servlet을 실행한다.
    - ApplicationFilterChain 구현체
        - Filter들이 chain.doFilter() 를 통해서 FilterChain을 계속 호출하게 된다.
        - ApplicationFilterChain은 현재 필터 위치가 어디인지 확인하고, 모든 Filter를 통과한게 확인되면 그때 서블릿을 실행한다.
        - 덕분에 Filter가 중간에 doFilter를 호출하지 않으면, 서블릿은 실행되지 않는다. (서킷 브레이커 성격)
        
        ```java
        @Override
        public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
            if (pos < n) {
                ApplicationFilterConfig filterConfig = filters[pos++];
                Filter filter = filterConfig.getFilter();
                filter.doFilter(request, response, this);
                return;
            }
            servlet.service(request, response);
        }
        ```
        
        - 이를 `Chain Of Responsibility Pattern` 패턴이라고도 부른단다.
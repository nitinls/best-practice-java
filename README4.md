@Bean
    @ConditionalOnProperty(name = "api-interceptor.enabled", havingValue = "true", matchIfMissing = false)
    public FilterRegistrationBean interceptorFilterRegistration() {
        final FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new ApiInterceptorFilter());
        registration.setServletNames(asList("CXFServlet"));
        registration.setName("ApiInterceptorFilter");
        registration.setOrder(-1);
        return registration;
    }

    @Bean
    @ConditionalOnProperty(name = "cros-sharing.enabled", havingValue = "true", matchIfMissing = false)
    public CrossOriginResourceSharingFilter cors() {
        return new CrossOriginResourceSharingFilter();
    }

    @Bean
    @ConditionalOnProperty(name = "cors-filter.enabled", havingValue = "true", matchIfMissing = false)
    public FilterRegistrationBean corsFilterRegistration() {
        final FilterRegistrationBean registration = new FilterRegistrationBean();
        final CorsHttpServletFilter filter = new CorsHttpServletFilter();
        registration.setFilter(filter);
        registration.setServletNames(asList("CXFServlet"));
        registration.setName("CorsFilter");
        registration.setOrder(3);
        return registration;
    }

    private static final class CorsHttpServletFilter implements Filter {

        private static final String CORS_ORIGIN_HEADER = "access-control-allow-origin";
        private static final String CORS_CREDENTIALS_HEADER = "access-control-allow-credentials";
        private static final String CORS_METHODS_HEADER = "access-control-allow-methods";
        private static final String ALL_WILDCARD = "*";
        private static final String TRUE_VALUE = "true";
        private static final String METHODS_VALUE = "GET, POST";

        @Override
        public void doFilter(final ServletRequest req, final ServletResponse res, final FilterChain chain)
                throws IOException, ServletException {

            chain.doFilter(req, res);

            if (res instanceof HttpServletResponse) {

                final HttpServletResponse response = (HttpServletResponse) res;

                final String corsOriginHeader = response.getHeader(CORS_ORIGIN_HEADER);

                if (StringUtils.isBlank(corsOriginHeader)) {

                    response.setHeader(CORS_ORIGIN_HEADER, ALL_WILDCARD);
                    response.setHeader(CORS_CREDENTIALS_HEADER, TRUE_VALUE);
                    response.setHeader(CORS_METHODS_HEADER, METHODS_VALUE);
                }
            }
        }

        @Override
        public void init(final FilterConfig config) throws ServletException {
            // do nothing
        }

        @Override
        public void destroy() {
            // do nothing
        }

    }
    

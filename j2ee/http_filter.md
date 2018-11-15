public void doFilter(ServletRequest request, 
                     ServletResponse response, 
                     FilterChain chain)
    throws IOException, ServletException
{
    boolean authorized = false;
    if (request instanceof HttpServletRequest) {
        HttpSession session = ((HttpServletRequest)request).getSession(false);
        if (session != null) {
            User user = (User) session.getAttribute("user");
            if (user != null)
                authorized = true;
        }
    }
            
    if (authorized) {
        chain.doFilter(request, response);
        return;
    } else if (filterConfig != null) {
        String login_page = filterConfig.getInitParameter("login_page");
        if (login_page != null && !"".equals(login_page)) {
            filterConfig.getServletContext().getRequestDispatcher(login_page).
     forward(request, response);
            return;
        }
    }
    
    throw new ServletException
    ("Unauthorized access, unable to forward to login page");
    
}


check if session is null , redirect to root page
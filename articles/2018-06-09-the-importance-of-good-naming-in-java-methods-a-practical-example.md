# The importance of good naming in Java methods - a practical example

(by Paolo Predonzani, originally posted on June 9, 2018)

I recently had a chat with [Carlo Bonamico](https://github.com/carlobonamico), an advocate
of [clean code design](https://github.com/carlobonamico/clean-code-design-principles-in-action)
and he pointed out how the ability to give good names in code is a strong indicator of a
coder's skills. I agreed and made a mental note to myself to spend a bit more
effort next time I choose the name of a class or method.

But it wasn't until today that this thing about names clicked with me. I was reviewing a
SecurityInterceptor class, which is responsible for ensuring authentication and
CSRF protection in a web application. Here's the original code:

```java
@Override
public boolean preHandle(
    HttpServletRequest request,
    HttpServletResponse response,
    Object handler
) throws Exception {
    String requestURI = urlPathHelper.getLookupPathForRequest(request);
    log.debug("requestUri: {}", requestURI);

    addResponseHeaders(response);

    HttpSession httpSession = request.getSession();
    UserSession userSession =
            (UserSession) httpSession.getAttribute(USER_SESSION_ATTRIBUTE);

    // Check if authentication is required
    if (userSession == null && requiresAuthentication(requestURI)) {
        if ("GET".equals(request.getMethod())) {
            redirectToLogin(request, response, requestURI);
        } else {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        }
        return false;
    }

    // Check if CSRF protection is required
    if (requiresCsrfProtection(requestURI)) {
        String csrfToken = request.getHeader(CSRF_TOKEN_HEADER);
        if (userSession == null ||
                !userSession.getCsrfToken().equals(csrfToken)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }
    }

    return true;
}

public boolean requiresAuthentication(String requestURI) {
    return requestURI.startsWith("/app") || requestURI.startsWith("/api");
}

public boolean requiresCsrfProtection(String requestURI) {
    return requestURI.startsWith("/api");
}
```

## First observation and refactoring

At first, this code didn't seem to have a naming problem. The preHandle method 
overrides a library method on which I have no control, while requiresAuthentication and
requiresCsrfProtection are simple and relatively self-explanatory.

No, the initial problem with preHandle() for me was that it was doing too many things
in one method:

* getting the request URI
* adding some response headers
* retrieving the user session
* checking authentication
* checking CSRF protection

I can imagine that if someone asked me in a few years what the method did, I would have
to read the lines one by one, examine the multiple "if" statements carefully, reconstruct
the logic and ultimately reconstruct the *intention* of the method.

Talking of intention, I noticed that I was trying to express it in the two comments
`// Check if...`:

```java
    // Check if authentication is required
    if (userSession == null && requiresAuthentication(requestURI)) {
        ...
    }
    
    // Check if CSRF protection is required
    if (requiresCsrfProtection(requestURI)) {
        ...
    }
```

There must be thousands of comments like this in the code I've written in my career.
It's almost a habit for me to write a little comment at each step in a method,
when the method does many steps, like in the example. I've never felt entirely
comfortable with this kind of comments, always wondering if the comment is of real value
or, rather, a repetition of what the code already says.

So here was the idea: what if I turned the comment into code. The comment was about
checking *if* authentication is required and the comment was just above an *if* statement.
The translation of comment to code appeared in my mind as something like:
`if (isAuthenticationRequired(...)) {/* do something */}`. So I extracted a method
and turned the comment into the method's name.


```java
@Override
public boolean preHandle(
    HttpServletRequest request,
    HttpServletResponse response,
    Object handler
) throws Exception {
    ...
    
    if (isAuthenticationRequired(userSession, requestURI)) {
        if ("GET".equals(request.getMethod())) {
            redirectToLogin(request, response, requestURI);
        } else {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        }
        return false;
    }

    ... CSRF stuff does something similar here ...
    
    return true;
}

private boolean isAuthenticationRequired(UserSession userSession, String requestURI) {
    return userSession == null && requiresAuthentication(requestURI);
}
```

A side note: at this point I wasn't trying to do anything smart with method names, as
the title of this post suggests. This awareness came later. Bear with me.
I was still simply trying to clean up a long, messy method.

## Second observation


Despite the refactoring, I wasn't happy yet. One reason was that the abundance
of "if's", "return false" and "return true" statements that make the logic difficult to understand:

```java
    if (isAuthenticationRequired(userSession, requestURI)) {
        if ("GET".equals(request.getMethod())) {
            redirectToLogin(request, response, requestURI);
        } else {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        }
        return false;
    }

    ... CSRF stuff does something similar here ...
    
    return true;
```

Reading the method in plain English, it
sounded like "If some condition is true, return false. Otherwise return true."

Somehow the logic "if true, return false" was suspicious. I'd rather have
"if true, return true", if only for aesthetic reasons.

Another reason for the unhappiness were the side effects in the "if" block.
In some cases the user is redirected to a login page, in other cases an SC_UNAUTHORIZED
error is produced, quite different things from returning a simple true or false value.

The "if's", the "return true/false" and the side effects were intertwined in a way
that looked impossible to untangle or to refactor.

## A turning point

Under these different pressures, the following question popped in my mind: what is the
*meaning* of the boolean that the preHandle() method returns? What is preHandle() "trying
to say"?

The answer was incredibly simple: preHandle() tells if everying is ok from a security
perspective. That's it. That is was that returned boolean means at a high level. True
means everything is ok. False means there is a problem of some kind.

If we look at the code, it's easy to get bogged down into the details of the if/then/else
logic, the side-effects, and the overall complexity. It is only by stepping back
and looking at things at a higher level that an unexpected simplicity appears.

So what does "is ok from a security perspective" mean? In my case it meant two things:

* authentication is ok
* CSRF protection is ok

Luckly, I could express this high level concept in just one final line of of the
preHandle() method:

```java
public boolean preHandle(
    ...
) throws Exception {
	/* bla bla omitted */
	
    return isAuthenticationOk(...) && isCsrfOk(...);
}
```

This felt like the right direction to pursue. The high-level meaning was nicely expressed
by the two methods `isAuthenticationOk()` and `isCsrfOk()`.

This is also the moment when I realised that good naming goes hand in hand with good
understing of the problem. By good naming I mean naming that expresses the code's
high-level intention and purpose, rather than the low-level functionality.

Sure, I still had to write those two methods and deal with all the redirect/error
side-effects, but those now looked like secondary problems.

Here is what I wrote for `isAuthenticationOk()`:

```java
private boolean isAuthenticationOk(
        HttpServletRequest request,
        HttpServletResponse response,
        UserSession userSession,
        String requestUri
) throws IOException {
    if (isAuthenticationRequired(userSession, requestURI)) {
        if ("GET".equals(request.getMethod())) {
            redirectToLogin(request, response, requestURI);
        } else {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        }
        return false;
    }
    
    return true;
}
```

I later changed the above code to avoid nested "if" statements. I think it improved
readability:

```java
private boolean isAuthenticationOk(
        HttpServletRequest request,
        HttpServletResponse response,
        UserSession userSession,
        String requestUri
) throws IOException {
    if (userSession != null) {
        return true;
    }

    if (!isAuthenticationRequired(requestUri)) {
        return true;
    }

    if ("GET".equals(request.getMethod())) {
        redirectToLogin(request, response, requestUri);
    } else {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
    }
    return false;
}
```

Notice that `isAuthenticationOk()` retains all the ugliness of the side-effects (redirect
and error) but at least the ugliness is contained in a clerly defined place.

## Putting it all together

The final code is the following. 

```java
@Override
public boolean preHandle(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler
) throws Exception {
    String requestUri = urlPathHelper.getLookupPathForRequest(request);
    log.debug("requestUri: {}", requestUri);

    addResponseHeaders(response);

    HttpSession httpSession = request.getSession();
    UserSession userSession =
            (UserSession) httpSession.getAttribute(USER_SESSION_ATTRIBUTE);

    return isAuthenticationOk(request, response, userSession, requestUri) &&
            isCsrfOk(request, response, userSession, requestUri);
}

//--------------------------------------------------------------------------
// Authentication-related methods
//--------------------------------------------------------------------------

private boolean isAuthenticationOk(
        HttpServletRequest request,
        HttpServletResponse response,
        UserSession userSession,
        String requestUri
) throws IOException {
    if (userSession != null) {
        return true;
    }

    if (!isAuthenticationRequired(requestUri)) {
        return true;
    }

    if ("GET".equals(request.getMethod())) {
        redirectToLogin(request, response, requestUri);
    } else {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
    }
    return false;
}

private boolean isAuthenticationRequired(String requestURI) {
    return requestURI.startsWith("/app") || requestURI.startsWith("/api");
}

//--------------------------------------------------------------------------
// CSRF-related methods
//--------------------------------------------------------------------------

private boolean isCsrfOk(
        HttpServletRequest request,
        HttpServletResponse response,
        UserSession userSession,
        String requestUri
) {
    if (!isCsrfProtectionRequired(requestUri)) {
        return true;
    }

    String csrfToken = request.getHeader(CSRF_TOKEN_HEADER);
    if (userSession != null && userSession.getCsrfToken().equals(csrfToken)) {
        return true;
    }

    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
    return false;
}

private boolean isCsrfProtectionRequired(String requestURI) {
    return requestURI.startsWith("/api");
}
```

## Conclusions

This post documents my "aha" moment as to using good names in code. Before, I used to
think of good naming as a kind of favor to the guy or girl who is going maintain
my code in 5 years. Since that moment though, I think of good naming and good understing
as inseparable.

Sometimes good understanding leads to good naming, sometimes the search for good naming
produces a better understanding. The lack of good naming may even be seen as "code smell",
as an indicator of problems that require attention.

This is certainly an area that I'd like to expand and experiment with in my next projects.



---
title: How clang-tidy works 🧹🧹🧹
date: 2022-08-10 10:32:00
tags: [compiler]
---

[clang-tidy](https://releases.llvm.org/11.0.0/tools/clang/tools/extra/docs/clang-tidy/index.html) is used to automatically fix common mistakes in C++ source code
(or to emit warnings, at least).
Similar tools in other languages are called "linter", they are often embedded into the language and/or standartized (like [PEP8](https://pep8.org/) for Python).

clang-tidy can find a number of bugs, outdated code, suspicious code patterns. It has a very big [list of checks](https://releases.llvm.org/11.0.0/tools/clang/tools/extra/docs/clang-tidy/checks/list.html).
For example, one may use the check to prevent uneffective `push_back`s when using vectors: [link to the check](https://releases.llvm.org/11.0.0/tools/clang/tools/extra/docs/clang-tidy/checks/performance-inefficient-vector-operation.html).
The user can choose which checks they want to use and tweak settings for each check.

Despite the fact that there are already almost 300 checks, you can still come up with a new idea for a checks.

### ✏️ Description of the check

I came up with a new check idea. How people used to define constant variables prior to C++17?
There was the pattern:
```c++
// in the .h file (declaration)
extern const std::string DVCC_DVVC_BLOCK_TYPE_NAME;

// in the .cpp file (definition)
const std::string DVCC_DVVC_BLOCK_TYPE_NAME = "Dolby Vision configuration";
```

In fact, you can just define the constant variable in the header file, and it will work, but it will be harsh, because
const variables are static by default. It means that every `.cpp` file that includes the header will deal with a unique
variable 👻
```c++
// this is bad! in the .h file (definition)
const std::string DVCC_DVVC_BLOCK_TYPE_NAME = "Dolby Vision configuration";
```

Starting from C++17 it is possible to properly define the variable in the header file:
```c++
// this is good! in the .h file (definition)
inline const std::string DVCC_DVVC_BLOCK_TYPE_NAME = "Dolby Vision configuration";
```

So my new check ought to find two bad usecases (as I presented to you: pre-C++17 and static variable) and suggest to use the third usecase (with the `inline` keyword).

### ✏️ How the check works

I sent the patch in February ([link to the pull request](https://reviews.llvm.org/D118743)), but haven't made it to the prod yet, because the review is slow
(they do it roughly one time per three months).

Let's look into the code to see how it works⚙️ Firstly we require C++17 or above:
```c++
  bool isLanguageVersionSupported(const LangOptions &LangOpts) const override {
    return LangOpts.CPlusPlus17;
  }
```

Clang parses the source code into the [AST](https://releases.llvm.org/8.0.0/tools/clang/docs/IntroductionToTheClangAST.html) (Abstract Syntax Tree).
The clang-tidy checks work on the AST level. They use [AST Matchers](https://releases.llvm.org/8.0.0/tools/clang/docs/LibASTMatchers.html) to find
the right nodes. AST Matchers library is easy to use, but it lacks some features and it is under active development.

If there is no AST Matcher to find nodes we are interested in, we can just write a new AST Matcher.

So, using the AST Matchers, let's elaborate what nodes we need.

These nodes should be variable declarations, namely *global constant non-inline variables at the file level* (that is, not inside the class).

Is it enough? Well, not exactly... If we carefully read the Standard (like we have an eternity to do it), turns out we should not
consider *variables inside anonymous namespaces* (it's useless to correct these variables), *template variables*
(they are implicitly inline), *volatile variables* (frankly I forgot why is that), and variables inside `extern "C"` as well. So, combining
all of this, we get this matcher:
```c++
  auto NonInlineConstVarDecl =
      varDecl(hasGlobalStorage(),
              hasDeclContext(anyOf(translationUnitDecl(),
                                   namespaceDecl())), // is at file scope
              hasType(isConstQualified()),            // const-qualified
              unless(anyOf(
                  isInAnonymousNamespace(), // not within an anonymous namespace
                  isTemplateVariable(),     // non-template
                  isInline(),               // non-inline
                  hasType(isVolatileQualified()), // non-volatile
                  isExternC()                     // not "extern C" variable
                  )));
```

Then register the matcher to find the two bad use-cases. The one use-case is **extern declarations** (we can mix AST matchers):
```c++
    Finder->addMatcher(varDecl(NonInlineConstVarDecl, isExternallyVisible())

                           .bind("extern-var-declaration"),
                       this);
```

The other use-case is **non-inline definitions**:
```c++
    Finder->addMatcher(varDecl(NonInlineConstVarDecl, isDefinition(),
                               unless(isExternallyVisible()))
                           .bind("non-inline-var-definition"),
                       this);
```

Then build warnings in the code that inspect the matched nodes:
```c++
  const VarDecl *D = nullptr;
  StringRef Msg;
  bool InsertInlineKeyword = false;

  if ((D = Result.Nodes.getNodeAs<VarDecl>("non-inline-var-definition"))) {
    Msg = "global constant %0 should be marked as 'inline'";
    InsertInlineKeyword = true;
  } else {
    D = Result.Nodes.getNodeAs<VarDecl>("extern-var-declaration");
    Msg = "global constant %0 should be converted to C++17 'inline variable'";
  }
```

If we see that the variable is declared not in a header file, then we do nothing (this cannot be caught yet at the AST Matchers level).

Then we can emit a nice warning at the place where the variable is declared. If there is the use-case with a non-inline definition, we
can fix the source code, implacing `"inline "` before the variable definition, so that clang-tidy could fix the code:
```c++
  DiagnosticBuilder Diag = diag(D->getLocation(), Msg) << D;
  if (InsertInlineKeyword)
    Diag << FixItHint::CreateInsertion(D->getBeginLoc(), "inline ");
```

That's all folks! Now you know how the statical code analysis tool works 🙂

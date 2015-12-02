---
layout: post
title: Unreal Engine 4 and C++ Notes
tags: gamedev, cpp
---

A list of notes and snippets for Unreal Engine 4 and C++.

clang-format
------------

Visual Studio 2015 is incapable of formatting UE4 code properly because of the macros.
[`clang-format`][1] can format the code properly, but will mess with the order of your
includes. The problem is, UE4 **requires** that the `<filename>.generated.h` include file
to be the last include. So, place this `.clang-format` file in the
`<Game Folder>/Source` directory:


{% highlight python %}
BasedOnStyle: LLVM
IndentWidth: 2
IncludeCategories:
  - Regex:           '.*(generated)'
    Priority:        10
  - Regex:           '.*'
    Priority:        1
{% endhighlight %}

The `IncludeCategories` entry is the important part. It tells the order of the include.
The `<filename>.generated.h` file will always be placed last because it will have a
higher priority number.


Visual Studio is bad?
--------------------

![vs-slow][2]

Is it familiar to you?.. it was too me :(

Solved it the extreme way by combining vim + emacs (I understand this is not for everyone).
[`Spacemacs`][3] to the rescue...

Here is my modifications to the `.spacemacs` file so far:

{% highlight lisp %}
;; Enable the C-C++ layer, but open all the headers, even ".c",
;; in C++-mode.
(defun dotspacemacs/layers ()
  (setq-default
   dotspacemacs-configuration-layers
   '(
     (c-c++ :variables
            c-c++-default-mode-for-headers 'c++-mode)
     )
  )
)

(defun dotspacemacs/user-config ()
  ;; Bind <TAB> to "clang-format" when in C++ mode
  ;; (works on selections)
  (add-hook 'c++-mode-hook 'clang-format-bindings)
    (defun clang-format-bindings ()
      (define-key c++-mode-map [tab] 'clang-format-buffer))

  ;; Show line numbers by default
  (global-linum-mode)
)
{% endhighlight %}

I keep both Visual Studio and Spacemacs open when coding. Spacemacs is useful
when editing files, and when I change a file, it is automatically updated in Visual
Studio. If I ever need to use Intellisense or something that Visual Studio has,
then I can just switch windows and use it. Also, compilations and debug are handled
inside Visual Studio too.


Print To Screen
---------------

{% highlight cpp %}
// inside your top header
#define PRINTSCR(text) if (GEngine) GEngine->AddOnScreenDebugMessage(-1, 3, FColor::White,text)
{% endhighlight %}

enum
----

{% highlight cpp %}
UENUM(BlueprintType)
enum class ETowerType : uint8 {
  VE_Solar UMETA(DisplayName = "Solar"),
  VE_Cube UMETA(DisplayName = "Cube"),
  VE_Battery UMETA(DisplayName = "Battery")
};

// usage
Type = ETowerType::VE_Cube;
{% endhighlight %}


Direct link to UPROPERTY, UFUNCTION, UCLASS properties
-------

 * [UPROPERTY, Properties specifiers][4]
 * [UFUNCTION, Functions specifiers][5]
 * [UCLASS, Class specifiers][6]

What the `T` in `TArray` means? `F` in `FText`? etc...
------------

Classes starting with:

 * `T` means they are templated classes
 * `U` means they are derived from `UObject`
 * `A` means they are derived from `AActor`
 * `I` are interfaces
 * `F` are everything else?

Pure Virtual functions
--------

Unreal requires that every class must be instantiable, so you can't
declare pure virtual functions. But, Unreal provides the `PURE_VIRTUAL` 
macro to help. Check its implementation:


{% highlight cpp %}
#define PURE_VIRTUAL(func,extra) { LowLevelFatalError(TEXT("Pure virtual not implemented (%s)"), TEXT(#func)); extra }
{% endhighlight %}

It requires two parameters. The first one should be the name of your function so it is easier to debug.
The second one is some extra code placed inside the function the macro is creating. Use it to return
a compatible value with your function return type. Check the example:


{% highlight cpp %}
UCLASS()
class UFoo : UObject {
  GENERATED_BODY()

public:
  virtual void RefreshDamage() PURE_VIRTUAL(UFoo::RefreshDamage, );
  
  virtual bool IsOkay() PURE_VIRTUAL(UFoo::IsOkay, return false);
}

UCLASS()
class UBar : UFoo {
  GENERATED_BODY()

public:
  void RefreshDamage() override; // use the "override" keyword
  bool IsOkay() override;
}
{% endhighlight %}



Functions != Methods
---------------

You can't use `UFUNCTION` with functions, as in, a function not inside a class.
If you want to make a native "function" callable from Blueprints, use a
static method:


{% highlight cpp %}
UCLASS()
class POWERTD_API UMathUtils : public UObject {
  GENERATED_BODY()

public:
  UFUNCTION(BlueprintCallable, Category = Math)
  static int32 GiveMeTheAnswer() { return 42; }
};
{% endhighlight %}




Structs
--------

{% highlight cpp %}
USTRUCT(Blueprintable, BlueprintType)
struct FTileIndex {
  GENERATED_USTRUCT_BODY()

  FTileIndex();
  FTileIndex(const int32 X, const int32 Y);

  UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Math)
  int32 X;
  UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Math)
  int32 Y;
};
{% endhighlight %}



Delegates and Blueprints
---------------------

Need more research.



 [1]: http://llvm.org/builds/
 [2]: /assets/images/vs_slow.png
 [3]: https://github.com/syl20bnr/spacemacs/
 [4]: https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/Reference/Properties/Specifiers/index.html 
 [5]: https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/Reference/Functions/Specifiers/index.html
 [6]: https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/Reference/Classes/Specifiers/index.html 

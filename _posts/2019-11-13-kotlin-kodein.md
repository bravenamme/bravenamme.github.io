---
layout: post
title:  "Kodein을 이용한 Kotlin 의존성 주입"
date:   2019-11-13 10:00
author: kapjong
tags:	[android, kotlin, kodein, Dependency_Injection]
---

# 의존성 이란?

네이버 사전 검색  
[명사]
1. 다른 것에 의지하여 생활하거나 존재하는 성질.
2. <약학> 금단 증상 때문에 계속하여 약물을 섭취하지 않으면 안 되는 상태.

개발적으로 단순의 정의 하자면 코드에서 두 모듈간의 연결을 의미 합니다.  
의존성이 크다는 것은 결합도가 높다는 것 이라고 해석 할수 있습니다.  

개발 시 하나의 모듈이 변경됨에 따라 결합된 다른 모듈이 영향을 받게 됩니다.  
결합도가 높은 프로젝트 모듈이 n개일 상황에 하나에 모듈 오류가 발생 할 경우 프로젝트 설정을 위해 밤을 지새워야 합니다.  
  
***모듈간 결합도가 높다면 독립성이 떨어지고 반대로 결합도가 낮다면 독립성이 높아 지게 됩니다.***

# 의존성 주입
의존성 주입은 두개이상의 모듈간의 결합도를 낮추기 위해 외부에서 객체를 생성하고 주입하는 것입니다.  
가장 큰 장점으로는 개발 진행 시 독립된 모듈에 개발 테스트 자유도가 보장 됩니다.  
개발 진행시 모듈간 의존성은 유지 하며 모듈간 독립성을 확보 하기 위하여 의존성 주입이 필요하게 되었습니다.

### ***안드로이드 대표 의존성 주입 콘퍼넌트***
- [dagger](https://github.com/square/dagger)  
- [Kodein](https://github.com/Kodein-Framework/Kodein-DI)

---
# [Kodein](https://kodein.org)
Kodein은 의존성들을 담는 일종의 컨테이너 입니다.  
모듈간 binding 처리와 및 의존성 주입을 도와 주는 도구 입니다.
  
안드로이드 java 환경에 의존성 주입에 경우 Dagger를 많이 사용 하지만  
Kotlin 환경에서 Kodein은 Dagger에 비해 가볍고 배우기도 사용하기도 쉽고 좀더 직관적입니다

### 사용 하기
```java
class MyApplication : Application() {
    override fun onCreate(savedInstanceState: Bundle?) {
        kodein.global.addModule(businessModule)
        kodein.global.addModule(androidModule(this))

        kodein.global.addConfiguration {
            bind<Controller>()
                with scoped(ActivityRetainedScope).singleton {
                    Controller(instance())
                }
        }
    }
}

class MyActivity : Activity(), KodeinGlobalAware {
    // retained between activity restarts
    val ctrl: Controller by instance()
}
```

---
## 하나에 모듈로 프로젝트 진행 중 인데 kodein을 사용해야 하나요?
> # 네

Kodein은 모듈간 bind 기능 외 객체 주입이 가능 하여 개발 시 큰 도움이 됩니다.

### KodeinAware 사용 하기
```java
class AppApplication() : Application(), KodeinAware {
    override val kodein by Kodein.lazy {
        import(androidXModule(this@AppApplication))
        bind<CompositeDisposable>() with instance(CompositeDisposable())
        bind<Markwon>() with instance(createMarkDown())
    }

    private fun createMarkDown(): Markwon {
        return Markwon.builder(this@AppApplication)
            .usePlugin(HtmlPlugin.create())
            .usePlugin(TablePlugin.create(this@AppApplication))
            // Glide instance
            .usePlugin(GlideImagesPlugin.create(this@AppApplication))
            // use supplied Glide instance
            .usePlugin(GlideImagesPlugin.create(Glide.with(this@AppApplication)))
            // Glide control
            .usePlugin(GlideImagesPlugin.create(object : GlideImagesPlugin.GlideStore {
                override fun cancel(target: com.bumptech.glide.request.target.Target<*>) {
                    Glide.with(this@AppApplication).clear(target)
                }
                override fun load(@NonNull drawable: AsyncDrawable): RequestBuilder<Drawable> {
                    return Glide.with(this@AppApplication).load(drawable.destination)
                }
            }))
            .build()
    }
}

open class Base_Activity : AppCompatActivity(), KodeinAware {
    override val kodein by closestKodein()
    val markwon: Markwon by instance()
}
```

---
## 참고
 * [dagger](https://github.com/square/dagger)
 * [Kodein-DI](https://github.com/Kodein-Framework/Kodein-DI)
 * [Kodein](https://kodein.org)
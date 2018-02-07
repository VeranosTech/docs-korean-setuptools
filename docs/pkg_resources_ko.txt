=============================================================
``pkg_resources``\ 를 이용한 패키지 탐색과 리소스 접근
=============================================================

``setuptools``\와 같이 배포된 ``pkg_resources`` 모듈은 파이썬 라이브러리가 자신의 리소스
파일에 접근하고 확장가능한 어플리케이션과 프레임워크가 플러그인을 자동적으로 탐색하게 하는
API를 제공한다. 또한 zip 파일 포맷 eggs 내에 있는 C 확장자 사용에 대한 런타임 지원과, 개별적으로
배포된 모둘 또는 서브패키지의 병합 지원, 그리고 활성 패키지의 현재 파이썬 "working set" 관리를
위한 API를 제공한다.


.. contents:: **목차**


--------
개요
--------

``pkg_resources`` 모듈은 설치된 파이썬 배포판을 탐색, 검토, 활성화, 사용하는 런타임 기능을
제공한다. 고급 기능 중 일부는 (특히, 여러 버전의 병렬 설치에 대한 지원) 명확하게 "egg" 포맷을
(zip 아카이브 또는 하위디렉토리로서) 필요로 한다. 반면에 플러그인 탐색 같은 다른 기능은 "egg-info"
메타데이터 디렉토리가 관련있는 배포판을 사용할 수 있는 한 정확하게 작동할 것이다.

Egg는 파이썬 모듈을 위한 배포 포맷으로 Java의 "jars"나 Ruby의 "gems", 또는 PEP 427로
정의된 "wheel"과 개념적으로 유사하다. 하지만 순수 배포 포맷과는 달리 eggs는 임포트 위치로
``sys.path``에 직접 추가되거나 설치될 수 있다. 이런 방식으로 설치되면 eggs 탐색가능하게 되며
이는 컨텐츠와 의존성을 명확하게 식별하는 메타데이터를 가진다는 의미다.
또한 설치된 egg는 자동적으로 검색되고 간단한 서식의 요청("docutils의 PDF 지원을 사용하기 위해서
필요한 모든 것을 가져다줘")에 대응해 ``sys.path``에 추가된다. 이 기능은 상호간에 충돌하는 배포판의
버전을 동일한 파이썬 환경에 공존할 수 있게 만들어주면서, ``sys.path``의 컨텐츠를 조작하는 식으로
개별적인 어플리케이션이 실행 시간에 필요한 버전을 작동하게 한다 (이것은 각 어플리케이션을 위한
분리된 환경을 생성하는 가상환경식 접근법과는 다르다).

아래의 용어들은 이 모듈에 의해 제공되는 기능을 설명하기 위하여 필요한 용어들이다:

project
    라이브러리, 프레임워크, 스크립트, 플러그인, 어플리케이션, 또는 데이터나 다른 리소스의 집합,
    또는 그것들의 집합. 프로젝트는 PyPI에 등록돼있는 이름처럼 "비교적 유일한" 이름을 가졌다고
    가정한다.

release
    특정한 시점의 프로젝트의 스냅샷, 버전 식별자로 표기.

distribution
    특정한 릴리즈를 나타내는 파일 또는 파일들.

importable distribution
    ``sys.path``에 위치해 있으면 파이썬이 안에 포함한 모듈을 임포트할 수 있게 해주는 파일이나
    디렉토리.

pluggable distribution
    파일 이름이 자신의 릴리즈(즉, 프로젝트와 버전)을 명확하게 식별하고 컨텐츠가 자신의 런타임
    요구 조건을 판족시키는 다른 프로젝트의 릴리즈를 명확하게 지정하는 임포트 가능한 배포판.

extra
    "extra"는 릴리즈의 선택적인 기능이며 추가적인 런타임 요구 조건을 부과한다.
    예를 들어, docutil PDF 지원이 존재하는 PDF 지원 라이브러리를 필요로 하면 docutils는
    자신의 PDF 지원을 "extra"로 지정하고 그 지원을 제공하기 위해 사용되어야 하는 다른 프로젝트
    릴리즈를 나열한다.

environment
    잠재적으로 임포트 가능하지만, 반드시 유효하지는 않는 배포판의 집합. 주어진 프로젝트를 위해서
    하나 이상의 배포판(릴리즈 버전)이 환경에 존재할 수도 있다.

working set
    ``sys.path``\ 에서 실제로 임포트 가능한 배포판의 집합. 주어진 프로젝트를 위해 하나의 배포판
    (릴리즈 버전)만이 환경에 존재할 수 있다.

eggs
    Eggs는 ``pkg_resources``에 의해 현재 지원되는 세 가지 포맷 중의 하나인 장작형
    배포판이다. built eggs, development eggs, egg links가 있다. Built eggs는
    이름이 egg naming 규칙을 따르고 ``EGG-INFO`` 하위 디렉토리(압축된)를 포함하는
    ``.egg``로 끝나는 디렉또리 또는 zip파일이다.Development eggs는 하나 이상의
    ``ProjectName.egg-info``하위 디렉토리가 있는 파이썬 코드로 이루어진 일반적인 디렉토리다.
    The development egg 포맷은 특정한 버전을 요청하기 위해 ``pkg_resources``\ 를
    사용하지 않는 소프트웨어에서 이용 가능한 기본 버전의 배포판을 제공하기 위해 사용되기도 한다.
    Egg links는 자체적 심벌릭 링크가 없는 (또는 심볼릭 링크 지원이 제한된) 플랫폼에서 심볼릭
    링크를 지원하는 built 또는 development egg의 이름을 포함하는 ``*.egg-link`` 파일이다.

(이 용어들과 개념에 대한 더 자세한 정보는, ``pkg_resources``\의 `architectural overview`_\ 와
일반적인 파이썬 eggs를 참고하라.)

.. _architectural overview: http://mail.python.org/pipermail/distutils-sig/2005-June/004652.html


.. -----------------
.. 개발자 가이드
.. -----------------

.. This section isn't written yet.  Currently planned topics include
    Accessing Resources
    Finding and Activating Package Distributions
        get_provider()
        require()
        WorkingSet
        iter_distributions
    Running Scripts
    Configuration
    Namespace Packages
    Extensible Applications and Frameworks
        Locating entry points
        Activation listeners
        Metadata access
        Extended Discovery and Installation
    Supporting Custom PEP 302 Implementations
.. For now, please check out the extensive `API Reference`_ below.


-------------
API 레퍼런스
-------------

네임스페이스 패키지 지원
=========================

네임스페이스 패키지는 자신만의 직접적인 컨텐츠가 없이 다른 패키지와 모듈만 포함하고 있는 패키지다.
이런 패키지는 여러 개의 패키지로 분리된 배포판으로 쪼개질 수 있다. 보통 이런 패키지는
한 조직에서 제작된 큰 패키지를 나눌 때 사용한다. 예를 들면, Zope Corporation packages를 위한
``zope`` 네임스페이스 패키지, Python Enterprise Application Kit을 위한 ``peak``
네임스페이스 패키지 등이 있다.

네임스페이스 패키지를 생성하기 위해서는 프로젝트의 ``setup.py``\ 에 있는 ``setup()``\ 에
``namespace_packages``\ 를 포함시켜야 한다. (더 자세한 정보는
:ref:`setuptools documentation on namespace packages <Namespace Packages>`\ 를
참고하라.) 또한, ``__init__.py``\ 파일에 ``declare_namespace()`` call을 추가해야 한다:

``declare_namespace(name)``
    입력된 패키지의 이름 `name`\ 을 포함된 패키지와 모듈이 여러 배포판으로 나누어질 수 있는
    "네임스페이스 패키지"라고 선언한다. 명명된 패키지의 ``__path__``\ 는 그 당시의 패키지를
    포함하는 ``sys.path``\에 있는 모든 배포 중에 일치하는 패키지를 포함시키도록 확장될 것이다.
    더 정확하게 말하자면 임포트 하는 사람의 ``find_module(name)``\ 는 loader를 반환하고
    패키지 컨텐츠를 위해 그것도 탐색이 될 것이다. 배포판의 ``activate()`` 메서드가 실행될 때마다
    그것은 네임스페이스 패키지의 존재를 확인하고 따라서 ``__path__``\ 컨텐츠를 업데이트 한다.

네임스페이스 패키지를 조작하거나 ``sys.path`` 를 실행시에 직접 바꾸는 어플리케이션은
이 API 함수를 사용해야 될 필요가 있다:

``fixup_namespace_packages(path_item)``
    `path_item`\ 이 존재하는 네임스페이스 패키지를 업데이트하기 위해 사용될 필요가 있는
    ``sys.path``에 새롭게 추가된 아이템이라고 선언한다. 보통 이 함수는 egg가 자동적으로
    ``sys.path``\ 에 추가됐을 때 호출된다. 만약 당신의 어플리케이션이 네임스페이스 패키지의
    일부를 포함하는 위치를 포함시키기 위해``sys.path``\ 를 수정하면, 존재하는 네임스페이스
    패키지에 추가됐는지 확인하기 위해 이 함수를 호출할 필요가 있다.

기본적으로 ``pkg_resources``\ 만 파일시스템과 zip importers를 위해 네임스페이스 패키지를
지원한다, 그리고 당신은 ``register_namespace_handler()``\ 을 사용하는 PEP 302와 호환되는
다른 "importers"로 지원을 확대할 수 있다.
See the section below on `Supporting Custom Importers`_ for details.


``WorkingSet`` 객체
======================

``WorkingSet`` 클래스틑 "유효한" 디스트리뷰션 집합에 접근할 수 있게 해준다. 일반적으로
의미있는 ``WorkingSet`` 인스턴스는 하나다: 그 인스턴스가 ``sys.path``\ 에서 현재 유효한
디스트리뷰션을 나타낸다. 이 전역 인스턴스는 ``pkg_resources`` 모듈 내의 ``working_set`` 이름
하에서 이용 가능하다. 그러나 전문 도구는 ``sys.path``와 일치하지 않는 working set 조작하려고
할 수 있다. 그래서 다른 ``WorkingSet`` 인스턴스를 생성하려고 할 수 있다.

전역 ``working_set`` 객체가 ``pkg_resources``\ 가 처음 임포트 될 때 ``sys.path``\ 로부터
초기화 되지만 ``pkg_resources``API를 통해서 모든 미래의 ``sys.path`` 조작을 다 한다면 업데이트만
된다는 사실을 주의하라. 만약 수동으로 ``sys.path``\ 를 수정하면 동기화를 유지하기 위해서
``workig_set`` 인스턴스에서 적절한 메소드를 불러와야 한다. 불행하게도 파이썬 ``sys.path``
같은 리스트 오브젝트에서 일어난 임이의 변화를 감지하는 방법을 제공해주지 않는다. 그래서
``pkg_resources``\ 는 ``sys.path``\ 의 변동을 기반으로 자동적으로 ``working_set``\ 을
업데이트 해주지 않는다.

``WorkingSet(entries=None)``
    반복가능한 경로 엔트리로부터 ``WorkingSet``\ 을 생성한다. 만약 `entries`\ 가 입력되지
    않았으면 컨스트럭터가 호출될 당시의 ``sys.path`` 값을 디폴트로 설정한다.

    일반적으로 ``WorkingSet`` 인스턴스를 직접 구성하는 일은 일반적으로 없지만 대신에
    암시적으로나 명시적으로 전역 ``working_set`` 인스턴스를 사용할 것이다. 대부분의 경우
    ``pkg_resources`` API는 ``working_set``\ 가 기본으로 사용되도록 제작되어서
    대부분의 시간동안 그것을 명시적으로 언급할 필요가 없다.

``sys.path``\ 에서 바로 이용 가능한 모든 디스트리뷰션은 ``pkg_resources``\ 가 임포트될 때
자동적으로 활성화 될 것이다. 이 동작은 어플리케이션의 버전 충돌을 일으킬 수 있는데 디폴트가 아닌
버전의 디스트리뷰션을 요구하게 된다. 이 상황을 리하기 위해서 ``pkg_resources``\ 는
디폴트 working set을 초기화 할 때 ``__main__`` 모듈에 있는 ``__requires__`` 특성을
확인하고 각각의 영향을 받는 디스트리뷰션의 적합한 버전을 활성화시킨다. 예시::

    __requires__ = ["CherryPy < 3"] # pkg_resources를 임포트하기 전에 설정되어야 한다.
    import pkg_resources


기본 ``WorkingSet`` 메소드
----------------------------

아래의 ``WorkingSet`` 객체 메소드들은 디폴트 ``working_set`` 인스턴스에 적용할 수 있는
``pkg_resources`` 에 있는 모듈 레벨의 함수로도 사용이 가능하다. 따라서, 예를 들면
pkg_resources.require()``\ 를 ``pkg_resources.working_set.require()``\ 의
축약형으로 사용할 수 있다:


``require(*requirements)``
    `requirements`\ 와 일치하는 배포판이 활성화 된다.

    `requirements`\ 는 반드시 스트링이나 (가능한 네스팅된) 스트링의 시퀀스여야 하며
    필요로 하는 디스트리뷰션과 버전을 지정해야 한다. 반환하는 값은 요구 조건을 이행하기
    위해서 활성화될 필요가 있는 디스트리뷰션의 시퀀스다; 이 working set에서 이미 활성화
    되었더라도 관련된 모든 디스트리뷰션은 포함되어 있다.

    요구 조건 지정자의 신택스는 아래에 있는 `Requirements Parsing`_\ 을 참고하라.

    일반적으로, 이 메서드를 직접 부를 필요는 없다. 이것은 제작용보다 약식 스크립팅과 양방향
    인터프리터 해킹을 위한 용도다. 만약 당신이 실제 라이브러리나 어플리케이션을 만든다면
    ``setuptools``\ 를 사용해서 "setup.py" 스크립트를 생성하고 그곳에 모든 요구 조건을
    선언해놓는 것을 적극적으로 권장한다. 그런 방식을 따르면 EasyInstall 같은 툴은 자동적으로
    당신의 패키지가 어떤 요구조건을 가지고 있는지 감지하고 거기에 맞춰서 처리할 수 있다.

    ``SomePackage``\ 가 이미 존재한다면 ``require('SomePackage')``\ 를 호출해도
    ``SomePackage``\ 를 설치하지 않을 것이다. 설치할 필요가 있으면 대신 ``resolve()``
    메서드를 (로컬 머신에서 필요한 디스트리뷰션이 찾아지지 않을 때 ``installer`` 콜백을
    전달하는 메서드) 사용해야 한다. 그 다음에 이 콜벡이 대화상자를 표시하거나 자동적으로
    필요한 디스트리뷰션을 다운로드하거나 당신의 어플리케이션에 적합한 다른 모든 일들을 하게
    만들 수 있다. 아래에 있는 ``resolve()`` 메서드를 참고하라, 그리고 ``Environment``
    객체의 ``obtain()`` 메서드를 참고하라.

``run_script(requires, script_name)``
    `requires`\ 에 의해 지정된 디스트리뷰션을 위키시키고 그건ㅅ의 `script_name` 스크립트를
    실행시킨다. `requires`\ 는 반드시 요구조건 지지정자를 포함한 스트링이어야 한다.
    (신택스는 아래의 `Requirements Parsing`_\ 을 참고하라.)

    찾아지면 스크립트는 *the caller's globals*\ 에서 실행될 것이다. 왜냐하면
    이 메서드는 디스트리뷰션에 있는 "진짜" 스크립트를 위한 프록시로 작동해서 랩퍼 스크립트에서
    호줄될 의도였기 때문이다. 랩퍼 스크립트는 정확한 인수름 집어넣은 이 함수를
    불러내는 것 말고는 일반적으로 아무것도 할 필요가 없다.

    만약 스크립트 실행 환경에서 더 많은 조정이 필요하다면 ``Distribution``\ 의 메서드인
    ``run_script()``\ 를 사용하길 원할 것이다.

``iter_entry_points(group, name=None)`

    `name`\ 이 None이면, working set에 있는 모든 디스트리뷰션의 `group`의 모든 엔트리
    포인트를 산출하고 아니면 `group`과 `name` 모두와 일치하는 엔트리포인트만 산출한다.
    엔트리 포인트는 디스트리뷰션이 working set에 나타나는 순서대로 유효한 디스트리뷰션에서
    산출된다. (전역 ``working_set``\ 의 경우 ``sys.path``\ 에 리스트 되어있는 순서와 같다.)
    개별 디스트리뷰션에 의해 선언되는 엔트리 포인트 사이에서는 특별한 순서가 존재하지 않는다.

    자세한 정보는 아래의 `Entry Points`_ 섹션을 참고하라.


``WorkingSet`` 메소드와 특성
-------------------------------------

이 메소드들은 특정한 working set의 컨텐츠를 조작하거나 질의하기 위해 사용된다.
그래서 특정한 ``WorkingSet`` 인스턴스에서 반드시 명시적으로 호출되어야 한다:

``add_entry(entry)``
    경로 항목을 ``entries``\ 에 추가하고, 거기서 디스트리뷰션을 검색한다. 추가적인 항목을
    ``sys.path``\ 에 추가하고 전역 ``working_set``\ 가 변동을 반영하게 하고 싶을 때
    반드시 사용해야 한다. 이 메소드는 설치 중에 ``WorkingSet()``\ 컨스트럭터에 의해 호출될
    수도 있다.

    이 메소드는 경로 엔트리를 따르는 디스트리뷰션을 찾기 위해 ``find_distributions(entry,True)``\ 를
    사용하고 그것들을  `add()`` 한다. `entry`\ 는 이미 존재해도 ``entries`` 특성에
    항상 추가된다. (이것은 왜냐하면 ``sys.path``\ 가 한 번 이상 같은 값을 포함할 수 있고,
    ``entries`` 특성이 이 부분을 반영할 수 있어야 하기 때문이다.)

``__contains__(dist)``
    이 ``WorkingSet``\ 에서 `dist`가 유효하면 True. 주어진 프로젝트의 하나의
    디스트리뷰션만이 주어진 ``WorkingSet``\ 에서 유효하다.

``__iter__()``
    working set에서 중복되지 않은 프로젝트를 위한 디스트리뷰션을 산출한다. 산출 순서는
    항목의 경로 엔트리가 working set에 추가된 순서를 따른다.

``find(req)``
    `req` (``Requirement`` 인스턴스)와 일치하는 디스트리뷰션을 찾는다. `req`\ 에 의해 지정된
    버전 요구조건이 충족 되는 한 요청된 프로젝트의 유효한 디스트리뷰션이 있으면 그것을 반환한다.
    하지만 `req` 요구조건을 충족하지 *못* 하면서 프로젝트의 유효한 디스트리뷰션이 있으aus
    ``VersionConflict``\ 가 발생한다. 요청된 프로젝트의 유효한 디스트리뷰션이 없으면
    ``None``\ 이 반환된다.

``resolve(requirements, env=None, installer=None)``
    (재귀적으로) `requirements`\ 를 충족할 필요가 있는 모든 디스트리뷰션을 나열한다.

    `requirements`\ 는 반드시 ``Requirement`` 객체의 시퀀스여야 한다. 만약 입력되면,
    `env`\ 는 ``Environment`` 인스턴스여야 한. 만약 입력되지 않음연 ``Environment``\ 가
    working set의 ``entries``\ 로부터 생성된다. 만약 입력되면 `installer`\ 는 이미 설치
    되어있는 디스트리뷰션에 의해 충족되지 못 하는 각 요구조건과 함께 호출될 것이다; 그것은 반드시
    ``Distribution``\ 이나 ``None``\ 을 반환해야 된다. (`installer`에 대한 자세한 정보는
    아래 `Environment Objects`_\ 의 ``obtain()`` 메서드를 참고하라.)
    argument.)

``add(dist, entry=None)``
    `dist`\ 를 `entry`\ 와 연관된 working set에 추가한다.

    `entry`\ 가 지정되있지 않으면 ``dist.location``\ 이 디폴트로 설정된다. 이 루틴이 종료될
    때 `entry`\ 가 working set의 ``.entries``\ 의 끝에 추가된다 (아직 없는 경우).

    set에 유효한 디스트리뷰션을 아직 프로젝트의 경우에 `dist`\ 는 working set에 추가만 된다.
    성공적으로 추가되었으면 ``subscribe()`` 메서드에 등록된 모든 콜백이 호출된다.
    (아래의 `변경 알람 받기`_ 참고)

    Note: ``add()``\ 는 ``require()`` 메서드에 의해서 자동적으로 호출되서 당신이
    일반적으로 이 메서드를 직접 사용할 필요는 없다.

``entries``
    이 특성은 "그림자" ``sys.path``\ 를 나타내며, 주로 디버깅에 유용하다. 임포트에 문제를
    겪고 있다면 전역 ``working_set`` 객체의 ``sys.path`` 에 대한 ``entries``\ 를
    일치하는지 보기 위해 확인해야 한다. 만약 일치하지 않는 경우, 당신의 프로그램이
    ``working_set``\ 업데이트 하지 않고 ``sys.path``\ 를 조작하고 있다는 의미다.
    중요한 주석: 직접 이 특성을 조작하지마라! ``sys.path``\ 와 동일하게 설정한다고 해서 문제를
    해결해주지 않는다. 엔진 경고에 검은 테이프를 붙이는 것이 차를 고쳐준다고 믿는 것과 똑같다!.
    만약 이 특성이 ``sys.path``\ 싱크가 맞지 않는다면 이것은 단지 문제가 있다는 *표시*\ 지
    문제의 원인이 아니다.


변경 알람 받기
------------------------------

확장가능한 어플리케이션과 프레임워크는 (플러그인 구성 요소 같은) 새로운 디스트리뷰션이 working set에
추가되었을 때 알림을 받을 필요가 있다. 이런 경우를 위해``subscribe()`` 메서드와 ``add_activation_listener()``
함수가 있다.

``subscribe(callback)``
    현재 set에 있거나 나중에 추가될 각각의 유효한 디스트리뷰션에 대해 한 번씩
    ``callback(distribution)``\ 를 호출한다. 왜냐하면 콜백이 이미 유효한 디스트리뷰션에
    대해 호출되기 때문에 있는 항목들을 처리하기 위해 working set에서 루프를 돌릴 필요는 없다;
    콜백을 등록하고 이 메서드에 의해 즉시 호출 된다는 사실을 대비해야 한다.

    콜백은 절대 예외가 전파되는 것을 허용해서는 안된다, 만약 전파되면 다른 콜백 작업을 방해해서
    woriking set 상태의 일관성을 없애버릴 수 있다. 무시하거나 로그를 남기고 또는 에러를
    처리하기 위해서, 특히 콜백을 호출하는 코드가 콜백 자신보다 에러 처리를 잘 하지 못할 때
    콜백은 try/except 블럭을 써야 한다.

``pkg_resources.add_activation_listener()`` is an alternate spelling of
``pkg_resources.working_set.subscribe()``.


플러그인 찾기
----------------

확장성 있는 어플리케이션은 종종 엔트리포인트나 다른 메타데이터를 로드하고 싶은 플러그인 디렉토리 set이나
플러그인 디렉토리를 가지는 경우가 있다. ``find_plugins()`` 메서드가 충돌이나 요구조건 누락 없이
로드 될 수 있는 최신 버전 프로젝트를 위한 환경을 스캔함을써 이러한 일을 가능하게 해준다.

``find_plugins(plugin_env, full_env=None, fallback=True)``
   `plugin_env`\ 을 스캔하고 어떤 디스트리뷰션이 이 working set에 버전 충돌이나 요구 조건
   누락 없이 추가 될 수 있는지 식별한다.

   사용 예시::

       distributions, errors = working_set.find_plugins(
           Environment(plugin_dirlist)
       )
       map(working_set.add, distributions)  # sys.path에 플러그인 추가
       print "Couldn't load", errors        # 에러 출력

   `plugin_env`\ 는 프로젝트의 "plugin directory" 또는 디렉토리에 있는 디스트리뷰션만
   포함하고 있는 ``Environment`` 인스턴스가 될 것이다.
   `full_env`\ 가 입력되면 현재 이용가능한 모든 디스트리뷰션을 포함한 ``Environment``
   인스턴스가 될 것이다.

   `full_env`\ 입력되지 않으면 하나가 ``WorkingSet``\ 에서 자동적으로 생성되는데,
   이 메서드가 호출된다는 것은 ``sys.path``\ 에 있는 모든 디렉토리는 디스트리뷰션을 위해 스캔될 될 것이라는
   것을 의미한다.

   이 메서드는 요소가 2개인 튜플을 반환한다: (`distributions`, `error_info`),
   `distributions`\ 은 의존성을 해결하기 위해 필요한 다른 모든 디스트리뷰션과
   로드할 수 있는 `plugin_env`\ 에서 찾은 디스트리뷰션의 리스트다. `error_info`\ 는
   로드할 수 없는 플러그인과 발생한 에러를 설명하는 예외 인스턴스를 맵핑한 사전이다. 일반적으로
   에러는 ``DistributionNotFound`` 또는 ``VersionConflict`` 인스턴스가 될 것이다.

   대부분의 어플리케이션은 주로 ``pkg_resource``\ 에 있는 마스터 ``working_set`` 인스턴스에
   있는 메서드를 사용할 것이다. 그리고 즉시 반환된 디스트리뷰션을 working set에 추가해서
   sys.path에서 이용할 수 있게 될 것이다. 이것은 모든 엔트리 포인트의 탐색을 가능하게 하고
   모든 다른 메타데이터 트래킹이나 훅을 활성화한다.

   ``find_plugins()``\ 에서 사용되는 해결 알고리즘은 다음을 따른다. 첫째,
   `plugin_env`\ 에 존재하는 디스트리뷰션의 프로젝트 이름은 분류된다.
   그 다음 각 프로젝트의 egg는 내림차숫 버전 순서로 시도된다 (즉, 최신버전이 먼서 시도된다).

   시도든 각 egg의 의존성을 해결하기 위해 이루어진다. 만약 시도가 성공하면, egg와 egg의 의존성은
   출력 리스트와 , working 일시적인 복사본에 추가된다. 해결 프로세스는 다음 프로젝트 이름으로
   계속 되고 해당 프로젝트의 오래된 egg는 시도되지 않는다.

   그러나 해결시도가 실패하면 에러가 에러사전에 추가된다. `fallback` 플래그가 참이면, 다음으로
   오래된 버전의 플러그인이 시도되고 작동ㅎ아는 버전을 찾을 때까지 계속된다. 실패하면
   해결 프로세스는 다음 플러그인 이름으로 해결 과정을 계속한다.

   몇몇 어플리케이션은 더 엄격한 대체 요구조건을 가지고 있다. 예를 들면, 데이터베이스 스키마와
   영속 객체를 가지고 있는 어플리케이션은 패키지의 버전을 안전하게 다운그레이드 할 수가 없을 것이다.
   다른 사람들은 새로운 플러은 설정이 100% 좋은지 확인하거나 좋다고 알려진 설정으로 돌아가는 것을
   원할 수 있다. (즉, `error_info` 반환 값이 비어있지 않으면 알려진 설정으로 되돌리고 싶어
   할 것이다.)

   이 알고리즘은 버전 충돌이 일어난 경우 알파벳 순으로 우선하는 프로젝트 이름의 의존성을 만족시키는
   데에 우선권을 부여한다. 만약 두 프로젝트의 이름이 "AaronsPlugin", "ZekesPlugin"이고
   둘 다 "TomsLibrary"의 다른 버전을 필요로 하면 "AaronsPlugin"이 이기고 "ZekesPlugin"은
   버전 충돌로 인해서 사용중단될 것이다.


``Environment`` 객체
=======================

"environment" 는 ``Distribution``\ 의 집합으로, 현재 플랫폼에 있고 잠재적으로 임포트가
가능하다. ``Environment`` 객체는 의존성 해결 시에 사용 가능한 디스트리뷰션을 인덱스 하기 위해
``pkg_resources``\ 에 의해 사용된다.

``Environment(search_path=None, platform=get_supported_platform(), python=PY_MAJOR)``
    `platform`, `python`\ 과 호환 가능한 디스트리뷰션의 `search_path`\ 를 스캐닝 함으로써
    환경 스냅샷을 생성한다. `search_path`\ 는 ``sys.path``\ 에서 사용되는 것 같은
    문자열의 시퀀스여야 한다. 만약 `search_path`\ 가 입력되지 않으면 ``sys.path``\ 가
    사용된다.

    `platform`\ 은 선색적인 문자열로 플랫폼 지정 디스트리뷰션이 반드시 호환해야 하는 플랫폼의
    이름을 지정한다. 만약 지정되지 않으면 현재 플랫폼을 디폴트로 설정한다. `python`\ 는
    선택적인 문자열로 권장되는 파이썬 버전을 지정한다; 현재 실행중인 버전을 디폴트로 설정한다.

    만약 현재 구동중인 플랫폼이나 파이썬 버전과 호환흔 것뿐만 아니라 *모든* 디스트리뷰션을 포함시키고
    싶으면 명시적으로 `platform` \ (과/또는 `python`)을 ``None``\ 으로 설정하면 된다.

    `search_path`\ 는 디스트리뷰션을 위해 즉시 스캔된다. 그리고 결과로 나온
    ``Environment``\ 는 찾아진 배포판의 스냅샷이다. 디스트리뷰션의 설치, 제거로 인해 시스템의
    상태가 변화하면 이것은 자동적으로 업데이트 된다.

``__getitem__(project_name)``
    주어진 프로젝트 이름에 있는 디스트리뷰션의 리스트를 반환하며 순서는 최신부터 가장 오래된
    버전으로 되어있다. (그리고 형식 우선순위는 같은 버전의 프로젝트를 포함하고 있는 디스트리뷰션이
    더 높다.) 프로젝트의 디스트리뷰션이 없으면 빈 리스트를 반환한다.

``__iter__()``
    환경에 있는 디스트리뷰션의 유일한 프로젝트 이름을 산출한다. 산출된 이름은 항상 소문자로
    나온다.

``add(dist)``
    생성시에 지정된 파이썬 버전, 플랫폼과 일치하면 환경에 `dist`\ 를 추가한다. 디스트리뷰션이
    아직 추가되지 않았을 때에만 추가한다. (즉, 한 번 이상 같은 디스트리뷰션이 추가되는 것은
    안 된다.)

``remove(dist)``
    환경에서 `dist`\ 를 제거한다.

``can_add(dist)``
    이 환경에서 `dist`\ 가 허용되는가? 환경이 생성되었을 때 지정된 ``platform``, ``python``
    버전과 호환되지 않으면 false 값이 반환된다.

``__add__(dist_or_env)``  (``+`` operator)
    ``Environment`` 인스턴스에 환경이나 디스트리뷰션을 추가하고 *새로운* 환경 객체를 추가하며
    객체는 둘 모두에 의해 이전에 포함되어 있던 모든 디스트리뷰션을 포함한다. 새로운 환경은 ``platform``\ 과
    ``None``\ 의 ``python``\ 을 가지고 있으며, 추가되는 어떠한 디스트리뷰션도 거절하지 않는
    다는 것을 의미한다; 단순히 추가되는 무엇이든 다 승인한다. 플랫폼과 파이썬 버전을 위해
    추가되는 아이템을 거르고 싶거나 *같은* 환경 인스턴스에 아이템을 추가하고 싶으면,
    대신 in-place 덧셈 (``+=``)\ 을 사용하라.

``__iadd__(dist_or_env)``  (``+=`` operator)
    디스트리뷰션이나 환경을 *가동중인* ``Environment`` 인스턴스에 추가하며 존재하는 인스턴스를
    업데이트하고 인스턴스를 반환한다. ``platform``, ``python`` 필터 특성이 효력이 있다. 그래서
    적합한 플랫폼 문자열이나 파이썬 버전을 가지고 있지 않은 소스에 있는 디스트리뷰션은 무시된다.

``best_match(req, working_set, installer=None)``
    `req`\ 가장 일치하고 ``working_set`\ 에서 사용할 수 있는 디스트리뷰션을 찾는다.

    적합한 디스트리뷰션이 활성화되어 있는지 확인하기 위해서 `working_set`의 ``find(req)``
    메서드를 호출한다. (특정한 `working_set`\ 에서 적합하지 않은 버전의 프로젝트가 이미
    활성화되어 있으면 ``VersionConflict``\ 를 일으킬 수 있다.) 적합한 디스트리뷰션이
    활성화되어있지 않으면 이 메서드는 환경에서 `req`\ 의 ``Requirement``\ 를 충족하는
    새로운 디스트리뷰션을 반환한다. 적합한 디스트리뷰션이 찾아지지 않고 `installer`\ 가
    입력되면 환경의 ``obtain(req, installer)`` 메서드 호출 결과가 반환될 것이다.

``obtain(requirement, installer=None)``
    요구조건과 일치하는 distro를 얻는다 (예, 다운로드를 통함). 기본 ``Environment`` 클래스에서
    이 루틴은 `installer`\ 가 None이 아니면, ``installer(requirement)``\ 를 반환하고
    None이면 None을 반환한다. 이 매서드는 하위 클래스가 `installer` 인수로 복귀하기 전에
    디스트리뷰션을 얻기 위한 다른 방법을 시도하도록 허용하는 훅이다.

``scan(search_path=None)``
    `platform`\ 에서 사용할 디스트리뷰션의 `search_path`\ 를 스캔한다.

    찾아진 모든 디스트리뷰션을 환경에 추가한다. `search_path`\ 는 ``sys.path``\ 에서
    사용되는 것 같은 문자열의 시퀀스여야 한다. 입력되지 않으면 ``sys.path``\ 가 사용된다.
    초기화에서 정의된 플랫폼/파이썬 버전을 따르는 디스트리뷰션만이 추가된다. 이 메서드는
    `search_path`\ 에 있는 항목으로부터 디스트리뷰션을 찾고 각각을 환경에 추가하기 위해
    ``add()``\ 를 호출하는 ``find_distributions()`` 함수의 축약형이다.


``Requirement`` 객체
=======================

``Requirement`` 객체는 몇몇 목적에 적합한 프로젝트의 버전을 표현한다. 이 객체 (또는
문자열 형식)는 스크립트나 디스트리뷰션이 필요로 하는 디스트리뷰션을 찾기 위해 다양한
``pkg_resources`` API에서 사용된다.


요구조건 파싱
--------------------

``parse_requirements(s)``
    문자열이나 반복가능한 라인인 ``Requirement`` 객체를 산출한다. 각 요구조건은
    반드시 새 행으로 시작해야 한다. 아래쪽의 신택스를 참고하라.

``Requirement.parse(s)``
    문자열이나 반복가능한 라인으로부터 ``Requirement``를 생성한다. 문자열이나 라인이
    유효한 요구조건 지정자를 포함하고 있지 않거나 두 개 이상의 지정자를 포함하고 있으면
    ``ValueError``\ 를 발생시킨다. (문자열이나 반복가능한 문자열에서 여러 지정자를 파싱하기
    위해서 ``parse_requirements()``\ 를 사용한다.)

    요구조건 지정자 신택스 전체는 PEP 508에 정의되어 있다.

    유효한 요구조건 지정자의 예시::

        FooProject >= 1.2
        Fizzy [foo, bar]
        PickyThing<1.6,>1.9,!=1.9.6,<2.0a0,==2.4c1
        SomethingWhoseVersionIDontCareAbout
        SomethingWithMarker[foo]>1.0;python_version<"2.7"

    이 프로젝트 이름은 요구 문자열에서 유일하게 필수적인 부분이며, 이것만 입력되면, 요구조건은
    모든 버전의 프로젝트를 받아들일 것이다.

    요구조건에서 "extras" 프로젝트의 선택적인 기능을 요청하기 위해 사용되며 작동하기 위해
    추가적인 프로젝트 디스트리뷰션을 필요로 할 것이다. 예를 들면, 가상의 "Report-O-Rama"
    프로젝트가 추가적인 PDF 지원을 제공한다면 그 지원을 제공하기 위해서 추가적인 라이프러리를
    필요로 할 것이다. 따라서 Report-O-Rama의 PDF 기능을 필요로 하는 프로젝트는 PDF 지원을
    제공하기 위해서 필요한 Report-O-Rama와 다른 라이브러리의 설치나 활성화를 요청하기 위해서
    ``Report-O-Rama[PDF]`` 의 요구조건을 사용할 수 있다. 예를 들어, 아래와 같이 사용할 수
    있다::

        easy_install.py Report-O-Rama[PDF]

    EasyInstall 프로그램을 사용하는 필요 패키지를 설치하거나 실행중에 sys.path에 디스트리뷰션을
    추가하는 ``pkg_resources.require('Report-O-Rama[PDF]')``\ 을 호출하기 위함이다.

    요구 조건의 "markers"는 요구 패키지가 설치되어야 할 때 지정하기 위해서 사용한다. 요구 조건은
    마커가 참이면 현재 환경에 설치될 것이다. 예를 들어, ``argparse;python_version<"2.7"``\
    로 지정하면 파이썬 2.7이나 3.3 환경에서는 설치 되지 않고 2.6 환경에서는 설치될 것이다.

``Requirement`` 메서드와 특성
--------------------------------------

``__contains__(dist_or_version)``
    `dist_or_version`\ 이 요구조건의 기준에 맞으면 True를 리턴한다. `dist_or_version`\ 이
    ``Distribution``\ 객체면, 프로젝트 이름은 요구조건의 프로젝트 이름과 일치하고, 버전은
    요구조건의 버전 기준을 충족해야 한다. `dist_or_version`\ 이 문자열이면
    ``parse_version()`` 유틸리티 함수를 이용해 파싱된다. 다른 경우 파싱이 된 버전으로
    간주된다.

    ``Requirement``\ 의 버전 지정자 (``.specs``)는 내부적으로 오름차순 순서로 분류되고
    받아들일 수 있는 버전의 범위를 정하기 위해 사용된다. 인접한 많은 조건은 효율적으로
    통합되고 (예, ``">1, >2"``\ 는 ``">2"``\ 와 같은 결과를 ``"<2,<3"`` 는 ``"<2"``\ 와
    같은 결과를 생산함), ``"!="``\ 버전은 범위 내에서 삭제된다. 적합성을 위해 테스트된 버전은
    범위 내의 멤버심에 대해 확인된다.

``__eq__(other_requirement)``
    대소문자를 구분하지 않는 동일한 프로젝트 이름, 버전 지정자, "extra"가 있으면
    요구조건은 다른 요구조건과 비교한다. (extras와 버전 지정자의 순서는 무시된다.)
    동일한 요구조건은 동일한 해쉬를 가지고 있어서, 요구조건은 집합이나 사전의 키로 사용될 수 있다.

``__str__()``
    ``Requirement``\ 의 문자열 형식은 ``Requirement.parse()``\ 에 전달되면
    동일한 ``Requirement`` 객체를 반환하는 문자열이다.

``project_name``
    필요한 프로젝트의 이름

``key``
    모든 소문자 버전의 ``project_name``, 비교나 색인에 유용함.

``extras``
    요구조건이 호출하는 "extras" 이름의 튜플. (모두 소문자가 되고 ``safe_extras()`` 파싱
    유틸리티 함수를 사용해 표준화돼서 요구조건이 생성되었던 extras와 정확히 동일하지 않을 수 있다.

``specs``
    ``(op,version)`` 튜플의 리스트, 오름차순 파싱된 버전 순서로 분류됨. 각 튜플의 `op`\ 는
    비교 연산자이며 문자열로 나타난다. `version`은 (파싱되지 않은) 버전 숫자다.

``marker``
    현재 환경에 대한 평가를 허용하는 ``packaging.markers.Marker`` 인스턴스. 마커 지정자가
    없을 경우에는 None이 됨.

``url``
    지정되면 요구조건을 다운로드 받을 위치.

엔트리 포인트
============

엔트리 포인트는 다른 디스트리뷰션에 의해 사용되는 함수나 클래스 같은 파이썬 객체를 디스트리뷰션이
"선언하기" 위해 사용되는 간단한 바업이다 확장가능한 어플리케이션과 프레임워크는
특정한 이름이나 그룸이 있는 엔트리 포인트를 특정한 디스트리뷰션이나 sys.path에 있는 모든 유효
디스트리뷰션 찾을 수 있고, 마음대로 선언된 객체를 불러오거나 조사한다.

엔트리 포인트는 파이썬 패키지나 모듈과 유사한 이름을 가진 "groups"에 속한다. 예를 들어
``setuptools`` 패키지는 ``distutils.commands``\ 이름의 엔트리 포인트를 사용하고
distutils 확장판으로 정의되는 커맨드를 찾기 위해서 사용된다. ``setuptools``\ 는 그룹에서
지정된 엔트리 포인트의 이름을 셋업 스크립트를 위해 허용 가능한 커맨드로 처리한다.

비슷한 방식으로, ``distutils.commands`` 같은 그룹에 있는 다이나믹 이름을 사용하건, 그룹에서
사전에 정의된 이름을 사용해서 다른 패키지는 자신의 엔트피포인트 그룹을 정의할 수 있다. 예를 들어
전, 후 퍼블리싱하는 다양한 훅을 제공하는 블로깅 프레임워크는 엔트리 포인트 그룹을 정의하고
그룹에 있는 "pre_process", "post_process" 이름의 엔트리 포인트를 찾을 수 있다.

엔트리 포인트를 선언하기 위해서 프로젝트는 ``setuptools``\ 를 사용하고
``entry_points`` 인수를 setup 스크립트에 있는 ``setup()``\ 에 제공해야 될 필요성이 있고
그래서 엔트리포인트는 디스트리뷰션의 메타데이터에 포함될 것이다. 더 자세한 정보는
``setuptools`` 문서를 참고하라.

각 프로젝트으 디스트리뷰션은 같은 엔트리 포인트 그룹에 있는 주어진 이름의 엔트리 포인트 중에 최대
하나를 선언할 수 있다. 예를 들어 다른 이름을 가지고 있는 한 distutils 확장은 두 개의 다른
``distutils.commands`` 엔트리 포인트를 선언할 수 있다. 그러나 같은 그룹에 있는 같은 이름의
엔트리 포인트를 광고하는 *다른* 프로젝트를 막을 수 있는 방법은 없다. 몇몇 경우에 이것은
바람직한 일이다. 왜냐하면 같은 엔트리 포인트를 사용하는 어플리케이션이나 프레임워크가 그것들을 훅으로
호출하거나 다른 경우에 그것들을 결합시킬 수 있기 때문이다. 여러 디스트리뷰션이 하나의 엔트리포인트를
선언하는 한다면 무엇을 할 것인지 결정하는 일은 어플리케이션이나 프레임워크에 달려있다; 두 엔트리
포인트를 사용하거나, 에러메세지를 표시하거나 sys 경로 순서에서 처음 발견된 것을 사용하는 것 등이
가능한 방법에 포함될 수 있다.


편의 API
---------------

다음의 함수에서 `dist`\ 인수는 ``Distribution``, ``Requirement`` 인스턴스나 요구조건을
지정하는 문자열이 될 수 있다 (프로젝트 이름, 버전 등). 인수가 문자열이거나 ``Requirement``\ 면
지정된 디스트리뷰션인 검색된다 (없으면 sys.path에 추가된다). 사용가능한 일치하는 디스트리뷰션이 없으면
에러가 발생한다.

`group` 인수는 엔트리 포인트 그룹을 식별하는 점으로 구분된 식별자를 포함하는 문자열이어야 한다.
만약 엔트리포인트 그룹을 정의하는 중이라면 다른 패키지의 엔트리 포인트 그룹과 충돌을 피하기 위해서
그룹 이름에 있는 당신의 패키지의 이름의 일부를 포함시켜야 한다.

``load_entry_point(dist, group, name)``
    지정된 디스트리뷰션으로부터 명명된 엔트리포인트를 불러오거나 ``ImportError``\ 를 일으킨다.

``get_entry_info(dist, group, name)``
    지정된 디스트리뷰션으로부터 주어진 `group`\ 과 `name`\ 에 대한 ``EntryPoint`` 객체를
    반환한다. 디스트리뷰션이 일치하는 엔트리포인트를 선언하지 않은 경우 ``None``\ 을 반환한다.

``get_entry_map(dist, group=None)``
    `group`\ 을 위한 디스트리뷰션의 엔트리 포인트 지도를 반환하거나 디스트리뷰션을 위한
    전체 엔트리 지도를 반환한다. 이 함수는 디스트리뷰션이 엔트리 포인트를 선언하지 않아도 항상
    사전을 반환한다. 만약 `group`\ 이 주어지면 사전은 엔트리 포인트 이름을 일치하는
    ``EntryPoint`` 객체에 맵핑한다. `group`\ 이 None이면 사전을 그룹 이름을 엔트리 포인트를
    그룹에 있는 일치하는 ``EntryPoint``인스턴스에 매핑하는 사전에 매핑한다.

``iter_entry_points(group, name=None)``
    `name`\ 에 매치되는 `group`\ 으로부터 엔트리 포인트 객체를 산출한다.

    `name`\ 이 None이면 sys.path에 있는 working set에 있는 모든 디스트리뷰션으로부터
    `group`\ 에 있는 모든 엔트리 포인트를 산출하고, 다른 경우 `group`\ 과 `name` 둘 다와
    일치하는 하나의 엔트리 포인트만을 산출한다. 엔트리 포인트는 유효한 디스트리뷰션이 sys.path에 나타나는
    순서대로 산출된다 (그러나, 틀정한 디스트리뷰션 내의 엔트리 포인트는 순서가 없다.)

    (이 API는 사실 전역 ``working_set`` 객체의 메서드다; 자세한 정보는 위 쪽에 있는
    `기본 workingset 메서드`_\ 를 참고하라.)


생성ㄱ와 파싱
--------------------

``EntryPoint(name, module_name, attrs=(), extras=(), dist=None)``
    ``EntryPoint`` 인스턴스를 생성한다. 엔트리포인트의 이름은 `name`\ 이다.
    `module_name`\ 은 선언된 객체를 포함하는 모듈의 (점으로 구분된) 이름이다. `attrs`\ 는
    선언된 오브젝트를 획득하기 위한 모듈로부터 검색하는 선택적인 이름 튜플이다.
    예를 들어 ``("foo","bar")``\ 의 `attrs`\ 와 ``"baz"``\ 의 `module_name`\ 은 선언된
    객체가 아래의 코드에 의해서 얻어질 수 있다는 의미다::

        import baz
        advertised_object = baz.foo.bar

    `extras`\ 는 이 엔트리 포인트를 제공하기 위해 디스트리뷰션이 필요로하는 "extra feature"
    이름의 선택적인 튜플이다. 엔트리 포인트가 로드되었을 때, 다른 디스트리뷰션이 sys.path에서
    활성화되기 위해서 필요한 것을 찾기 위해 extra 기능이 `dist` 인수에서 검색된다;
    자세한 정보는 ``load()`` 메서드를 참고하라. `extras` 인수는 `dist`\ 가 정의되었을 때만
    의미가 있다. `dist`\ 는 ``Distribution`` 인스턴스여야 한다.

``EntryPoint.parse(src, dist=None)`` (classmethod)
    문자열 `src`\ 로부터 단일 엔트리 포인트를 파싱한다

    엔트리 포인트 신택스는 아래의 형태를 따른다::

        name = some.module:some.attr [extra1,extra2]

    엔트리 이름과 모듈 이름이 요구된다, 그러나 ``:attrs``, ``[extras]`` 부분은
    항목 일부의 사이에 있는 공백처럼 선택 사항이다. `dist` 인수는 `src`\ 로부터 파싱된
    다른 값들과 함께 ``EntryPoint()`` 컨스트럭터로 전달된다.

``EntryPoint.parse_group(group, lines, dist=None)`` (classmethod)
    엔트리 포인트 이름을 ``EntryPoint`` 객체에 매핑하는 사전을 만들기 위헤 `lines`
    (문자열 또는 여러 줄의 시퀀스)을 파싱한다. 엔트리 포인트 이름이 중복되었거나 `group`\ 이
    유효하지 않은 엔트리 포인트 그룹 이름이거나 신택스 에러가 있을 경우 ``ValueError``\ 가
    발생한다. (`group` 파라미터는 검정과 더 상세한 에러메세지를 생성하기 위해서만 사용된다.)
    `dist`\ 가 제공되면, 생성된 ``EntryPoint`` 객체의 ``dist`` 특성을 설정하기 위해서
    사용될 것이다.

``EntryPoint.parse_map(data, dist=None)`` (classmethod)
    `data`\ 를 파싱해서 룹 이름을 엔트리 포인트 이름을 ``EntryPoint`` 객체에 맵핑한 사전에
    매핑하는 사전으로 만든다.`data`\ 가 사전이면, 키는 그룹 이름으로 사용되고 값은
    ``parse_group()``\ 에 `lines` 인수로 전단된다. `data`\ 가 문자열이거나 줄로 된
    시퀀스면 첫 번째는 .ini 스타일 섹션으로 분리되고 (``split_section``\ 을 사용) 섹션 이름은
    그룹 이름으로 사용된다. 어느 경우든 `dist` 인수는 ``parse_group()``\ 에 전달되어서
    엔트리 포인트는 지정된 디스트리뷰션에 링크될 것이다.


``EntryPoint`` 객체
----------------------

간단한 검사의 경우 ``EntryPoint`` 객체는 컨스트럭터 인수 이름과 정확히 일치하는 특성을 가지고 있다:
``name``, ``module_name``, ``attrs``, ``extras``, ``dist`` 모두 사용 가능하다.
추가적으로 아래의 메서드들도 제공된다:

``load()``
    엔트리 포인트를 로드하고, 선언된 파이썬 객체를 리턴한다. 실제로 ``self.require()``\ 을
    호출하고 ``self.resolve()``\ 를 반환한다.

``require(env=None, installer=None)``
    엔트리 포인트에 필요한 모든 "extras"가 sys.path에서 사용 가능한지 확인한다.
    ``EntryPoint``\ 가 ``extras``\ 를 가지고 있으면서 ``dist``\ 는 없거나  ,
    명명된 엑스트라가 디스트리뷰션에 의해 정의되지 않은 경우 ``UnknownExtra``\ 를 일으킨다.
    만약 `installer`\ 가 입력되면, ``Requirement`` 인스턴스를 취하고 일치하는 임포트 가능한
    ``Distribution`` 인스턴스나 None을 반환하면서 호출 가능해야 한다.

``resolve()``
    엔트리 포인트를 모듈과 특성으로부터 분리하고, 선언된 파이썬 오브젝트를 반환한다.
    얻어지지 않으면 ``ImportError``\ 를 발생시킨다.

``__str__()``
    ``EntryPoint``\ 의 문자열 형태는 동일한 ``EntryPoint`` 를 생성하기 위해
    ``EntryPoint.parse()``\ 에 전달될 수 있는 문자열이다.


``Distribution`` 객체
========================

``Distribution`` 객체는 파이썬 코드의 집합을 나타내며 코드는 임포트 되지 않을 수도 있고,
관련된 메타데이터와 자원을 가지고 있지 않을 수도 있다. 메타데이터는 다른 디스트리뷰션에 어떤 다른
프로젝트에 의존하고 있는지 디스트리뷰션이 선언하는 엔트리 포인트가 무엇인지 등의 정보를 포함하고 있다.


Distributions 생성 또는 획득하기
---------------------------------

일반적으로 당신은 `WorkingSet``\ 이나 ``Environment``\ 에서 ``Distribution`` 객체를
얻을 것이다. (위쪽의 `WorkingSet Objects`_ 섹션과 `Environment Objects`_ 섹션을 참고하라,
각각은 유효한 디스트리뷰션과 사용 가능한 디스트리뷰션의 컨테이너다.) 당신은 또한 아래의 고레벨 API
중에 하나에서 ``Distribution`` 객체를 얻을 수 있다:

``find_distributions(path_item, only=False)``
    `path_item`\ 을 통해서 접근 가능한 디스트리뷰션을 산출한다. 만약 `only`\ 가 참이면,
    ``location``\ 이 `path_item`\ 과 동일한 디스트리뷰션들만 산출한다. 즉, `only`\ 가
    참이면 이것은 ``sys.path``\ 에 `path_item`\ 이 있으면 임포트 할수 있는 모든 디스트리뷰션을
    다 산출한다. `only`\ 가 거짓이면 이것은 또한 `path_item` 안 또는 아래에 있는 디스트리뷰션을
    산출한다. 그러나 위치가 ``sys.path``\ 에 추가되지 않으면 임포트 되지 않는다.

``get_distribution(dist_spec)``
    주어진 ``Requirement`` 또는 문자열에 대한 ``Distribution`` 객체를 반환한다.
    만약 `dist_spec`\ 이 이미 ``Distribution`` 인스턴스면 그게 반환된다.
    만약 하나로 파싱될 수 있는 문자열이나 ``Requirement`` 객체면 이것은 일치하는 디스트리뷰션을
    찾고 활성화시키기 위해 사용되고 일치하는 디스트리뷰션이 리턴된다.

그러나 당신이 디스트리뷰션을 사용해서 작업하기 위한 특수한 툴을 제작하고 있거나
새로운 배포 포맷을 제작하고 있으면 아래의 세 가지 컨스트럭터 중 하나를 사용해 직접
``Distribution`` 객체를 생성할 필요가 있을 수 있다.

이 컨스트럭터들은 모두 디스트리뷰션과 관련된 메타데이터나 자료에 접근하기 위해 사용되는
선택적인 `metadata` 인수를 취한다. `metadata`\ 는 ``IResourceProvider`` 인터페이스를
실행하는 객체이거나 None이어야 한다. 만약 None일 경우 ``Emptyprovider``\ 이 대신 사용된다.
``Distributin`` 객체는 `metadata` 객체에 `IResourceProvider`_\ 와
`IMetadataProvider Methods`_\ 을 위임함으로써 둘 모두를 실행한다.

``Distribution.from_location(location, basename, metadata=None, **kw)`` (classmethod)
    url, 파일 이름, ``sys.path``\ 에서 사용되는 다른 문자열 형식의 `location`\ 을
    위한 디스트리뷰션을 만든다. `basename`\ 은 ``Foo-1.2-py2.4.egg``\ 같이
    디스트리뷰션의 이름을 정하는 문자열이다. `basename`\ 이 ``.egg``\ 로 끝나면
    프로젝트의 이름, 버전 파이썬 버전, 플랫폼은 파일이름에서 추출되고, 생성된
    디스트리뷰션의 각 특성으로 고정된다. 추가적인 키워드 인수는 ``Distribution()``
    컨스트럭터로 전달된다.

``Distribution.from_filename(filename, metadata=None**kw)`` (classmethod)
    로컬 파일 이름을 파싱해서 디스트리뷰션을 만든다.
    ``Distribution.from_location(normalize_path(filename),
    os.path.basename(filename), metadata)``\ 보다 짧은 방법이다. 즉, 위치가
    이름과 버전 정보를 파일 이름의 기저 부분에서 파싱해서 표준화된 형태의 파일 이름을
    가진 디스트리뷰션을 생성한다. 추가적인 키워드 인수는 ``Distribution()``
    컨스트럭터로 전달된다.

``Distribution(location,metadata,project_name,version,py_version,platform,precedence)``
    속성을 세팅해서 디스트리뷰션을 만든다. `py_version` (현재 파이썬 버전이
    디폴트 설정으로 되어있음)과 `precedence` (``EGG_DIST``\ 로 디폴트 설정; 자세한 정보는
    아래 `Distribution 특성`_ 참고) 인수를 제외한 모든 인수는 선택적 이고 디폴트는
    None으로 되어 있다. ``from_filename()``\ 이나, ``from_location()``
    컨스트럭터를 사용하는 것이 모든 인수를 개별적으로 지정하는 것보다 쉽다.


``Distribution`` 특성
---------------------------

location
    디스트리뷰션의 위치를 나타내는 문자열. 임포트 가능한 디스트리뷰션의 경우
    동적으로 임포트하기 위해서 ``sys.path``\ 추가된다. 임포트 되지 않는 디스트리뷰션의
    경우 파일이름, url, 위치를 나타내는 다른 형식이다.

project_name
    디스트리뷰션의 프로젝트 이름인 문자열. 프로젝트 이름은 프로젝트의 setup 스크립트를
    통해서 정의되고, PyPO 에서 프로젝트를 식별하기 위해서 사용된다. ``Distribution``\ 이
    구성되었을 때, `project_name`\ 은 ``safe_name()`` 유틸리티 함수를 통해 전달돼서
    허용되지 않는 문자를 걸러낸다

key
    ``dist.key``\ 는``dist.project_name.lower()``\ 의 축약형이다. 프로젝트 이름으로
    디스트리뷰션의 색인이나 대소문자를 구별하지않는 대조를 위해 사용된다.

extras
    문자열로 된 리스트, 프로젝트의 의존성 리스트(프로젝트의 setup 스크립트에 지정된
    ``extras_require`` 인수)에 정의된 추가 기능의 이름을 제공한다.

version
    이 디스트리뷰션이 프로젝트의 어떤 릴리즈를 포함하고 있는지 표시하는 문자열.
    ``Distribution`\ 이 구성되면 `version` 인수가 ``safe_version()`` 유틸리티
    함수를 통해서 전달돼서 허용되지 않는 문자를 걸러낸다. `version`\ 이
    구성될 때 지정되지 않으면 나중에 이 특성에 접근을 시도하면 ``Distribution``\ 이
    자신의 ``PKG-INFO`` 메타이데이터 파일을 읽어서 자신의 버전을 찾으려고 시도한다.
    ``PKG-INFO``\ 이 사용 불가능하거나 파싱될 수가 없으면 ``ValueError``\ 가
    발생한다.

parsed_version
    ``parsed_version``\ 는 디스트리뷰션의 ``version``\ 의 파싱된 형태를 나타내는
    객체다. ``dist.parsed_version``\ 는 ``parse_version(dist.version)``\ 를
    호출하는 쉬운 방법이다 디스트리뷰션을 버전에 따라 분류하거나 비교하기 위해 사용된다.
    (``parse_version()`` 함수에 대한 자세한 정보는 아래의 `Utilities 파싱하기`_
    섹션을 참고하라.) ``Distribution``\ 이 `version`\ 이나 손실된 버전 정보를
    전달해주는 `metadata` 없이 구성되었으면 ``parsed_version``\ 에 접근할 경우
    ``ValueError``\ 가 발생할 수 있다.

py_version
    문자열로서 디스트리뷰션이 지원하는 major/minor 파이썬 버전을 나타낸다.
    예를 들어, "2.7" 또는 "3.4". 디폴트는 현재 파이썬 버전이다.

platform
    디스트리뷰션이 목표로 했던 플랫폼을 나타내는 문자열, 디스트리뷰션이 "순수 파이썬"
    교차 플랫폼인 경우 ``None``. 플랫폼 문자열에 대한 자세한 정보는 아래의
    ``Platform Utilities`_\ 를 참고하라.

precedence
    디스트리뷰션의 ``precedence``\ 는 같은 `project_name``\ 과 ``parsed_version``\ 을
    가진 두 디스트리뷰션의 상대적 순서를 결정하기 위해 사용된다. 디폴트 우선은
    ``pkg_resources.EGG_DIST``\ 이며 가장 높은(즉, 가장 선호되는) 우선이다.
    사전 정의된 우선순위의 전체 리스트는, 가장 선호되는 것부터 최소로 선호되는 것
    순으로: ``EGG_DIST``, ``BINARY_DIST``, ``SOURCE_DIST``, ``CHECKOUT_DIST``,
    ``DEVELOP_DIST``\ 이다.  일반적으로, ``EGG_DIST`` 이외의 우선은 설치 적합성을
    결정하는 패키지 인덱스에서 검색된 디스트리뷰션을 분류할 때
    ``setuptools.package_index`` 모듈에 의해서만 사용된다. 그러나 "System"과
    "Development" eggs는 (즉 ``.egg-info``\ 포맷을 사용하는) ``DEVELOP_DISt``의
    우선권을 자동으로 부여받는다.



``Distribution`` 메서드
------------------------

``activate(path=None)``
    디스트리션이 `path`\ 에서 임포트 가능하게 한다. `path`\ 가 None이면 대신에
    ``sys.path``\ 가 사용된다. 디스트리뷰션의 ``location``\ 이 `path` 안에 있고,
    필요한 이름공간 패키지 수정과 선언을 수행한다. (즉, 디스트리뷰션이 이름 공간
    패키지를 포함하고 있으면, 이 메서드는 패키지를 선언하고 이름 공간 패키지를 위한
    디스트리뷰션의 내용을 다른 동적 디스트리뷰션에서 제공되는 컨텐즈와 병합한다.
    자세한 정보는 `Namespace Package SUpport`_\ 섹션을 참고하라.)

    ``pkg_resources``\ 는 전역 ``working_set``\ (디스트리뷰션이 추가될 때마다
    이 메서드가 호출된다)에 알림 콜백을 추가한다.  그러므로, 일반적으로 명시적으로
    이 메서드를 호출할 필요는 없다. (sys.path 에 있는 이름 공간 패키지는
    ``pkg_resource``\ 가 있는 한 항상 임포트된다. 이것 때문에 이름공간 패키지는
    스테이트먼트를 임포트하거나 코드를 포함해서는 안된다.)

``as_requirement()``
    이 디스트리뷰션의 이름과 버전이 일치하는 ``Requirement`` 인스턴스를 반환한다.

``requires(extras=())``
    디스트리뷰션의 의존성을 지정하는 ``Requirement`` 객체를 나열한다. `extras`\ 가
    지정되어 있으면 디스트리뷰션에 의해 지정된 "extras"의 이름 시퀀스여야 하며 반환된
    리스트는 "extras"를 지원하기 위해 필요한 의존성을 포함할 것이다.

``clone(**kw)``
    디스트리뷰션의 복사본을 생성한다. 입력되는 키워드 인수는 ``Distribution()``
    컨스트럭터에 대응하는 인수를 무시하고 복사된 디스트리뷰션의 특성 일부를 변경할 수
    있게 해준다.

``egg_name()``
    디스트리뷰션의 표준 파일 이름이 무엇이어야 하는지를 ".egg" 확장자를 포함하지 않고
    반환한다. 예를 들어 윈도우즈 파이썬 2.3에서 실행되는 버전 1.2 "Foo"  프로젝트
    디스트리뷰션은 ``Foo-1-2-py2.3-win32``\ 라는 ``egg_name()``\ 을 가질 것이다.
    이름이나 버전에 있는 대쉬는 언더스코어로 바퀸다. (``Distribution.from_location()``\ 는
    ".egg" 파일 이름을 파싱할 때 그것들을 다시 되돌린다.)

``__cmp__(other)``, ``__hash__()``
    디스트리뷰션 객체는 해쉬가 된 다음 파싱된 버전과 우선을 기초로 비교되고
    뒤에 키(소문자 프로젝트 이름), 위치, 파이썬 버전, 플랫폼이 온다.

아래의 메서드는 배포판으로 선전된 ``EntryPoint`` 객체에 접근하기위해 사용된다.
자세한 정보는 위쪽에 있는 `Entry Points`_ 를 참고하라:

``get_entry_info(group, name)``
    `group`\ 과 `name`\ 의 ``EntryPoint`` 객체를 반환하고, 디스트리뷰션에 의해
    선전된 포인트가 없으면 None이 반환된다.

``get_entry_map(group=None)``
    `group`\ 의 엔트리 포인트 맵을 반환한다.  `group`\ 이 None이면 그룹 이름을
    모든 그룹의 엔트피 포인트 맵에 대응시킨 사전을 반환한다. (엔트리 포인트 맵은
    ``EntryPoint``\ 에 대한 엔트리 포인트 이름의 사전이다.)

``load_entry_point(group, name)``
    ``get_entry_info(group, name).load()``\ 의 축약형. 명명된 엔트리 포인트에 의해
    선전된 객체를 반환한거나 엔트리 포인트가 디스트리뷰션에 의해 선전되지 않았거나
    다른 임포트 문제가 있을 경우 ``ImportError``\ 를 발생시킨다.

위의 메소드 외에 ``Distribution`` 객체는 `IResourceProvider`_,
`IMetadataProvider Methods`_\ 의 모든 메소드를 실행할 수 있다. (다음 섹션들에
설명되어 있다):

* ``has_metadata(name)``
* ``metadata_isdir(name)``
* ``metadata_listdir(name)``
* ``get_metadata(name)``
* ``get_metadata_lines(name)``
* ``run_script(script_name, namespace)``
* ``get_resource_filename(manager, resource_name)``
* ``get_resource_stream(manager, resource_name)``
* ``get_resource_string(manager, resource_name)``
* ``has_resource(resource_name)``
* ``resource_isdir(resource_name)``
* ``resource_listdir(resource_name)``

만약 디스트리뷰션이 `metadata` 인수를 가지고 생성됐으면, 이 리소스와 메다데이터
접근 메서드는 `metadata` 제공자에게 위임된다. 그렇지 않은 경우 ``EmptyProvider``\ 에
위임돼서 디스트리뷰션은 리소스나 메타데이터가 없이 나타날 것이다. 이 위임 방식은
커스텀 임포터를 지원하거나 적절한 `IResourceProvider`_ 구현을 생성해서
새로운 배포 포맷이 간단하게 작동되도록 하기 위해 사용된다; 아래의
`커스텀 임포터 지원하기`_\ 를 참고하라.


``ResourceManager`` API
=======================

``ResourceManager`` 클래스는 자원이 파일과 디렉토리로 존재하든 특정 종류로 압축 되었든
패키지 리소스에 대한 일관된 접근을 제공한다.

일반적으로, ``pkg_resources`` 모듈이 전역 인스턴스를 생성해주고, ``pkg_resources``
모듈 이름 공간에 있는 최상위 레벨의 이름으로 메서드의 대부분을 사용할수 있게 해주기
때문에``ResourceManager`` 인스턴스를 명시적으로 관리하거나 생성할 필요는 없다,
그래서 예를 들면, 이 코드는 실제로 전역 ``ResourceManager``\ 의 ``resource_string()``
메서드를 실제로 호출 한다::

    import pkg_resources
    my_data = pkg_resources.resource_string(__name__, "foo.dat")

따라서, 명시적인 ``ResourceManager`` 인스턴스 필요 없이 아래의 API를 사용할 수 있다;
그냥 임포트해서 필요한대로 사용하면 된다.


Basic Resource Access
---------------------

아래의 메서드에서 `pakage_or_requirement` 인수는 파이썬 패키지/모듈 이름 (예,
``foo.bar``) 또는 ``Requirement`` 인스턴스일 수 있다. 패키지 또는 모듈 이름인 경우
명명된 모듈이나 패키지는 임포트 되고 (즉, 디스트리뷰션이나 ``sys.path`` 에 있는
디렉토리 안에 있다), `resource_name` 인수는 명명된 패키지에 따라 해석된다.
(모듈 이름이 사용되었으면, 리소스 이름은 즉시 명명된 모듈을 포함하는 패키지와 연관된다.
또한 이ㅣ름공간 패키지 이름을 사용해서는 안 된다. 왜냐하면 이름 공간 패키지는 여러
디스트리뷰션에 걸쳐서 퍼져있고 리소르를 위해서 어떤 디스트리뷰션을 찾아야 하는지
모호해지기 때문이다.

``Requirement``\ 이면, 요구 조건이 자동적으로 분해되고(필요한 경우 현재 ``Environment``\ 를
검색한다) 존재하지 않으면 일치하는 디스트리뷰션이 ``WorkingSet``과 ``sys.paht``\ 에
추가된다. (``Requirement``\ 가 충족되지 않으면, 예외가 발생한다.) `resource_name`\ 는
식별된 디스트리뷰션의 루트에 따라 해석된다; 즉 첫 번째 경로 부분은 디스트리뷰션에 있는
최상위 레벨의 모듈이나 패키지의 피어로 처리될 것이다.

리소스 이름은 ``/``\ 로 분리된 경로여야 하고 절대경로 (즉, ``/``\ 로
시작하면 안 됨) 이거나 ``".."`` 같은 상대 이름을 포함해서는 안된다. 파일 시스템 경로가
아니기 때문에 ``os.path`` 루틴을 사용해서 리소스 경로를 조작하면 안 된다.

``resource_exists(package_or_requirement, resource_name)``
    명명된 리소스가 존재하는가? ``True`` 또는 ``False`` 반환한다.

``resource_stream(package_or_requirement, resource_name)``
    지정된 리소스에 대한 읽을 수 있는 파일 형태의 객체를 반환한다; 실제 파일,
    ``StringIO`` 이거나 유사한 객체일 수도 있다. 리소스에 있는 모든 바이트는
    그대로 읽혀진다는 점에서 스트림은 바이너리 모드다.

``resource_string(package_or_requirement, resource_name)``
    문자열로 지정된 리소스를 반환한다. 리소스는 바이너리 방식으로 읽혀지며, 반환된
    문자열은 정확히 리소스에 저장된 바이트를 포함하고 있다.

``resource_isdir(package_or_requirement, resource_name)``
    명명된 리소스가 사전인가? ``True`` 또는 ``False``\ 를 반환한다.

``resource_listdir(package_or_requirement, resource_name)``
    리소스가 zip파일 안에 있어도 작동한다는 점을 제외하면 ``os.listdir``\ 처럼 명명된
    리소스 사전의 컨텐츠를 나열한다.

리소스 타입에 관해서 ``resource_exists()``, ``resource_isdir()``\ 만 구분하지 않는다
파일 리소스에서 ``resource_listdir()``\ 을 사용할 수 없고, 디렉토리 리소스에서
``resource_string()``, ``resource_stream()``\ 를 사용할 수 없다. 리소스 타입에
부적절한 메서드를 사용하는 것은 플랫폼이나 포함된 디스트리뷰션의 포맷에 따라서 정의되지
않은 동작이나 예외를 일으킨다.

리소스 추출
-------------------

``resource_filename(package_or_requirement, resource_name)``
    종종 문자열이나 스트림 형태로 리소스에 접근하는 것으로 충분하지 않으며, 실제
    파일시스템 파일이름이 필요하다. 그러한 경우 리소스에 대한 파일 이름을 얻으려면
    이 메서드를 (또는 모듈 레벨의 함수) 사용해야 한다. 리소스가 압축된 egg 같은
    아카이브 디스트립뷰션이면, 캐쉬 디렉토리에 추가될 것이고 캐쉬에 있는 파일 이릉미
    반환될 것이다. 명명된 리소스가 디렉토리면, 그 디렉토리에 (하위 디렉토리 포함)있는
    모든 리소스 또한 추출된다. 명명된 리소스가 C 익스텐션이거나 "eager resource"면
    (``setuptools`` 도큐먼테이션 참고), 모든 C 익스텐션과 eager 리소스가 동시에
    추출된다.

    아카이브된 자원은 아래의 두 메서드에 의해 관뢰될 수 있는 캐쉬 위치로 추출된다:

``set_extraction_path(path)``
    필요한 경우, 리소스가 추출될 기본 경로를 설정한다.

    추출이 시작되기 전에 이 루틴을 호출하지 않으면 경로는 ``get_default_chache()``\ 의
    반환 값으로 디폴트 설정이 된다. (``PYTHON_EGG_CACHE`` 환경 변수를 기반으로 하고,
    다양한 플랫폼 한정 대체 폴백이 있다. 더 사제한 정보는 루틴의 도큐먼테이션을
    참고하라.

    리소스는 리소스 제공자에 의해 주어진 정보에 기반한 경로의 하위 디렉토리에
    추출된다. 임시 디렉토로 설정해도 되지만 끝났을 때 추출된 파일을 지워야 하므로
    반드시 ``cleanup_resorces()``\ 를 호출해야 한다.

    당신이 ``cleanup_resources()``\ 를 첫 번째로 호출하지 않으면 일단 리소스가
    추출되면 주어잔 자료 매니저를 위해 경로를 바꿀 수 없다.

``cleanup_resources(force=False)``
    추출된 모든 리소스 파일과 디렉토리를 삭제하고, 성공적으로 제거되지 않은
    디렉토리와 파일 이름 리스트를 반환한다. 이 함수는 일시처리 보호를 받지 않는다.
    따라서 일반적으로 추출 경로가 다일 프로세스에서 독점적으로 이루어지는 임시
    디렉토리일 때만 호출되어야 한다. 이 메서드는 자동적으로 호출되지 않는다;
    추출에 쓰였던 임시 디렉토리를 정리하고 싶으면 반드시 명시적으로 호출하거나
    ``atexit`` 함수로 등록해야 한다.


"Provider" 인터페이스
--------------------

새로운 배포 아카이브 포맷을 위해서 ``IResourceProvider``, ``IMetadataProvider``\ 을
구현하고 있는 중이면 자료 추출 기느을 파일 시스템과 결합시키기 위해서
``IResourceManager`` 메서드를 사용해야 한다. 그러나, 아카이브 포맷을 구현하지 않는다면
이 메서드를 사용할 필요는 없다. 앞에서 설명된 여러 메서드들과는 달리 이 메서드들은
전역 ``ResourceManager``\ 에 묶인 최상위 함수로서 사용할 수가 없다;
따라서 그 메서드들을 사용하려면 명시적인 ``ResourceManager`` 인스턴스가 있어야
한다.

``get_cache_path(archive_name, names=())``
    `archive_name`, `names`\ 에 관한 캐시에 있는 절대 위치를 반환한다

    존재하지 않는 경우 결과 경로의 부모 디렉토리가 생성될 것이다. `archive_name`\ 는
    외함하는 egg(외함하는 zip 파일의 이름이 아닐 수 있음)의 가저 파일이며, ".egg"
    확장자를 포함한다. `names`\ 가 제공되면 `names`\ 는 egg의 추출 위치 아래에 있는
    경로 이름 부분의 시퀀스여야 한다.

    이 메서드는 나중에 정리하기 위해서 생성된 이름을 추적하기 때문에, 추출 위치를
    얻어야 할 필요가 있는 리소스 제공자에 의해서, 추출될 예정인 이름에 대해서만
    호출되어야 한다.

``extraction_error()``
    추출 프로세스를 간섭하는 동적 예외를 설명해주는 ``ExtractionError`` 발생시킨다.
    캐시 경로에서 파일을 추출하는 중에 OS error가 발생하면 이 메서드를 호출해야 한다;
    이것은 운영체제 예외를 포맷해주고 다른 정보를, 추출 예외를 자체적으로 처리하고 래핑하는
    프로그램에서 필요로 하는 ``ExtractionError`` 인스턴스에 추가해준다.

``postprocess(tempname, filename)``
    `tempname`\ 의 지정 플랫폼 후처리를 수행한다. 리소스 제공자는 성공적으로 압축된
    리소스를 추출했을 때만 이 메서드를 호출해야 한다. 이미 파일 시스템에 있는
    리소스에 대해서는 호출하면 안 된다.

    `tempname`\ 파일의 임시 이름이고, 'filename'\ 은 이 루틴이 반환된 후에 호줄자에
    의해 다시 명명될 것이다.


메타데이터 API
=============

메타데이터 API는 접속가능한 디스트리뷰션에 묶여있는 메타데이터 리소스에 접근하기 위해
사용된다. 메타데이터 리소스는 가상 파일 또는 디렉토리로, "plugins"에 연결하는 확장
어플리케이션이나 프레임워크에 사용될 수 있는 디스트리뷰션에 관한 정보를 포함하고 있다.
다른 종류의 리소스와 마찬가지로, 메타데이터 리소스 이름은 ``/``\ 로 구분되어 있고
``..``\ 를 포함하거나 ``/``\ 로 시작하면 안 된다. 리소스 경로를 조작하기 위해서
``os.path`` 루틴을 사용해서는 안 된다.

메타데이터 API는 ``IMetadataProvider`` 또는 ``IResourceProvider`` 인터페이스를
시행하는 객체에 의해 제공된다. ``get_provider()`` 함수에 의해 반환되는 객체가
하는 것처럼, ``Distribution`` 객체가 이 인터페이스를 시행한다;

``get_provider(package_or_requirement)``
    패키지 이름이 입력되면, 패키지를 위한 ``IResourceProvider``\ 반환된다.
    ``Requirement``\ 이 입력력되면, 현재 (필요한 경우 현재의 ``Environment``\ 를
    찾고 새로 찾은 ``Distribution``\ 을 working set에 추가하는) working set으로부터
    ``Distribution``\ 을 반환함으로써 요구조건을 분해한다. 만약 명명된 패키지가
    임포트 되지 않거나 ``Requirement``\ 가 충족되지 않으면, 예외가 발생한다.

    주석: ``Requirement`` 대신 패키지 이름을 사용하면, 돌려받는 객체가 플러그
    가능한 디스트리뷰션이 아니게 될 수도 있으며 이는 설치하는 패키지의 방법에 따라
    다르다. 특히 "개발" 패키지와 외부 관리되는 단일 버전 패키지는 패키지 이름을
    대응하는 프로젝트의 메타데이터와 매핑할 수 있는 방법을 가지고 있지 않다. 패키지
    이름을 ``get_provider()`` 에 전달하고 반환되는 객체로부터 프로젝트 메타데이터를
    회수하는 코드를 쓰지 마라. 명명된 패키지가 ``.egg`` 파일이나 디렉토리에 있을 때
    작동이 되는 것처럼 보이지만, 다른 설치 시나리오에서는 실패할 것이다. 프로젝트
    메타데이터를 원하는 경우, 패키지가 아니라 *프로젝트*\ 에 요청할 필요가 있다.


``IMetadataProvider`` 메서드
-----------------------------

``IMetadataProvider`` 또는 ``IResourceProvider`` 인스턴스를 시행하는 오브젝트에서
(``Distribution`` 인스턴스 등) 제공되는 메서드는 아래와 같다:

``has_metadata(name)``
    지명된 메타데이터 리소스가 존재하는가?

``metadata_isdir(name)``
    지명된 메타데이터가 디렉토리인가?

``metadata_listdir(name)``
    (``os.litdir()`` 같은) 디렉토리에 있는 메타디에터 이름의 리스트.

``get_metadata(name)``
    지명된 메타데이터 리소스를 문자열로 반환함. 데이터는 바이너리 모드로 읽는다; 즉,
    리소스 파일의 정확한 바이트가 반환된다.

``get_metadata_lines(name)``
    빈 칸과 코멘트 라인이 없는 라인들의 리스트로 지명된 메타데이터 리소스를 산출한다.
    이것은 ``yield_lines(provider.get_metadata(name))``\ 의 생략형이다. 인식하는
    신택스에 대한 자세한 정보는 아래쪽의 `yield_lines()`_\ 에 있는 섹션을 참고하라.

``run_script(script_name, namespace)``
    입력된 이름 공간 사전에 있는 지명된 스크립트를 실행한다. ``script`` 메타데이터 디렉토리에
    그 이름을 가진 스크립트가 없으면 ``ResolutionError``\ 를 발생시킨다. `namespace`\ 는
    파이썬 사전이어야 하며 일반적으로 스크립트가 모듈로 실행되면 모듈 사전이어야 한다.


Exceptions
==========

``pkg_resources``\ 는 프로세스가 패키지를 찾고 가동시키는 요청을 할 때 일어날 수 있는 문제에 대한
간단한 예외 계층을 제공한다::

    ResolutionError
        DistributionNotFound
        VersionConflict
        UnknownExtra

    ExtractionError

``ResolutionError``
    이 클래스는 다른 세 가지 예외에 대해 기본 클래스로 사용되며, 단일 "예외" 구문으로
    모두를 캐치할 수 있다. 요청한 디스트리뷰션 내에 존재하지 않는 스크립트를 실행하려는
    시도를 하는 것 같은 여러 종류의 요구조건 해결 문제에 대해 발생할 수도 있다.

``DistributionNotFound``
    요구조건을 이행하기 위해 필요한 디스트리뷰션이 찾아지지 않는다.

``VersionConflict``
    요청된 버전의 프로젝트가 가동 중인 같은 프로젝트의 버전과 충돌한다.

``UnknownExtra``
    요청된 "extras" 중 하나가 요청한 디스트리뷰션에 의해 인식되지 않는다.

``ExtractionError``
    리소스를 파이썬 egg 캐시로 추출할 때 문제가 발생핬다. 아래의 특성들은 이 예외의
    인스턴스에서 사용 가능하다:

    manager
        예외를 발생시킨 리소스 관리자

    cache_path
        리소스 추출을 위한 기본 디렉토리

    original_error
        추출을 실패하게 한 예외 인스턴스


커스텀 importers 지원하기
===========================

기본적으로, ``pkg_resources``\ 는 일반 파일시스템 임포트와 ``zipimport`` importers를
지원한다. 다른 (PEP 302 호환) importers나 모듈 loaders로 ``pkg_resource`` 기능을
사용하고 싶으면 아래의 API를 사용하는 함수를 지원하고 다양한 handler를 등록해야 될
필요가 있다:

``register_finder(importer_type, distribution_finder)``
    ``sys.path`` 항목에 있는 디스트리뷰션을 찾는 `distribution_finder`\ 를 등록한다.
    `importer_type`\ 은 타입이거나 PEP 302 "Importer"(``sys.path`` 항목 handler)의
    클래스이며 `distribution_finder`\ 는 경로 항목, importer 인스턴스 또는 `only`
    플래그를 전달했을 때, 그 경로 항목 하에서 찾아지느 ``Distribution`` 인스턴스를
    산출한다. (만약 참이면, 'only' 플래그는 파인더가 오로지 ``location``\ 이 제공된
    경로 항목과 동일한 ``Distribution`` 객체만 산출한다는 의미다.

    finder 함수의 예시는 ``pkg_resources.find_on_path``\ 의 소스를 참고하라.

``register_loader_type(loader_type, provider_factory)``
    `loader_type`\ 를 위한 ``IResourceProvider``\ 를 만드는 `provider_factory`\ 를
    등록한다. `loader_type`\ 는 PEP 302 ``module.__loader__``\ 의 클래스나 타입이며
    `provider_factory`\ 는 모듈 객체를 전달했을 때 그 모듈을 위한 `IResourceProvider`_\ 를
    반환하며 `ResourceManager API`_\ 와 사용될 수 있게 해준다.

``register_namespace_handler(importer_type, namespace_handler)``
    제공된 `importer_type`\ 을 위한 이름공간 패키지를 선언하는 `namespace_handler`\ 를
    등록한다. `importer_type`\ 은 타입이거나 PEP 302 "Importer"(``sys.path`` 항목 handler)의
    클래스이며 `namespace_handler`\ 아래 같은 시그니처로 호출 가능하다::

        def namespace_handler(importer, path_entry, moduleName, module):
            # 자식 패키지에 사용하는 경로 엔트리를 반환한다.

    이름공간 handler는 관련된 importer 객체가 관련있는 경로 항목을 다루어 된다고 이미
    동의했으면 호출만 가능하다. handler는 모듈 ``__path__``\ 가 동일한 하위 경로를
    포함하고 있지 않으면 하위 경로만 반환해야하 한다. 그렇지 않은 경우 None을 리턴해야
    한다.

    이름공간 handler의 예시는 zip파일 임포트와 정규 임포트에서 사용되는
    ``pkg_resources.file_ns_handler`` 함수의 소스를 참고하라.


IResourceProvider
-----------------

``IResourceProvider``\ 는 ``register_loader_type()``\ 에 등록된 `provider_factory`\ 로
반환된 객체의 어떤 메소드가 필요한지 문서화한 추상적인 클래스다. ``IResourceProvider``\ 는
``IMetadataProvider``\ 의 하위 클래스다. 그래서 이 인터페이스를 시행하는 객체는 반드시
여기에 있는 메서드 뿐만 아니라 `IMetadataProvider Methods`_\ 의 모든 메서드를
시행해야 한다. 아래에 있는 메서드에 대한 `manager` 인수는 위에 있는 문서화된 전체
`ResourceManager API`_\ 를 지원하는 객체여야 한다.

``get_resource_filename(manager, resource_name)``
    리소스가 파일 시스템에 언팩되어야 한다면 `manager`\ 로 추출된 것을 조정하며,
    `resource_name`\ 에 대한 참인 파일시스템 경로를 반환한다.

``get_resource_stream(manager, resource_name)``
    `resource_name`\ 에 대한 읽을 수 있는 파일 같은 객체를 반환한다.

``get_resource_string(manager, resource_name)``
    `resource_name`\ 의 컨텐츠를 포함하는 문자열을 반환한다.

``has_resource(resource_name)``
    패키지가 지명된 리소스를 포함하고 있는가?

``resource_isdir(resource_name)``
    지명된 리소스가 디렉토리인가? 리소스가 존재하지 않거나 디렉토리가 아니면 false를
    반혼한다.

``resource_listdir(resource_name)``
    리소스 디렉토리의 컨텐츠의 리스트를 반환한다. 존재하지 않는 디렉토리의 컨텐츠를
    요청하는 것은 예외를 발생시킨다.

그런데 제공자 클레스는 ``IResourceProvider`` 또는 ``IMetadataProvider`` 하위 클래스로
분류할 필요가 없다 (하면 안된다). 이 클래스들은 오로지 문서화 목적으로 존재하며 유용한
시행 코드를 제공하지 않는다.  대신에 `built-in resource providers`_\ 의 하나를
하위 클래스로 분뷰하고 싶을 수도 있다.


빌트인 리소스 제공자
---------------------------

``pkg_resources``\ 는 적절한 곳에서 자동적으로 사용되는 몇 가지 제공자 클래스를
포함하고 있다. 상속 트리는 아래와 같다::

    NullProvider
        EggProvider
            DefaultProvider
                PathMetadata
            ZipProvider
                EggMetadata
        EmptyProvider
            FileMetadata


``NullProvider``
    이 제공자 클래스는 일반적인 제공자 작동(스크립트 실행 등)을 제공하는
    추상적인 기초이며 몇 가지 추상적인 메서드의 정의를 제공한다.

``EggProvider``
    제공자 클래스는 압축 되었거나 압축이 풀린 egg에 일반적인 egg 한정 기능에
    추가된다.

``DefaultProvider``
    이 제공자 클래스는 언팩된 egg와 "오래된 일반 파이썬" 파일시스템 모듈에서
    사용된다.

``ZipProvider``
    이 제공자 클래스는 모듈이 egg든 아니든 zip된 뮤듈에서 사용된다.

``EmptyProvider``
    이 제공자 클래스는 항상 메타데이터와 리소스가 없는 제공자와 일관된 답을
    반환한다. 대신에 ``Distribution`` 객체는 ``metadata`` 인수 사용 없이 이 제공자
    클래스의 인스턴스를 생성한다. 모든 ``EmptyProvider`` 인스턴스는 동일하기 때문에,
    하나의 인스턴스만 있으면 된다. 그러므로 ``pkg_resources``\ 는 ``empty_provider``
    이름 아래서 이 클래스의 전역 인스턴스를 생성하고 ``EmptyProvider`` 인스턴스가
    필요하면 이것을 사용할 수 있다. 

``PathMetadata(path, egg_info)``
    Create an ``IResourceProvider`` for a filesystem-based distribution, where
    `path` is the filesystem location of the importable modules, and `egg_info`
    is the filesystem location of the distribution's metadata directory.
    `egg_info` should usually be the ``EGG-INFO`` subdirectory of `path` for an
    "unpacked egg", and a ``ProjectName.egg-info`` subdirectory of `path` for
    a "development egg".  However, other uses are possible for custom purposes.

``EggMetadata(zipimporter)``
    Create an ``IResourceProvider`` for a zipfile-based distribution.  The
    `zipimporter` should be a ``zipimport.zipimporter`` instance, and may
    represent a "basket" (a zipfile containing multiple ".egg" subdirectories)
    a specific egg *within* a basket, or a zipfile egg (where the zipfile
    itself is a ".egg").  It can also be a combination, such as a zipfile egg
    that also contains other eggs.

``FileMetadata(path_to_pkg_info)``
    Create an ``IResourceProvider`` that provides exactly one metadata
    resource: ``PKG-INFO``.  The supplied path should be a distutils PKG-INFO
    file.  This is basically the same as an ``EmptyProvider``, except that
    requests for ``PKG-INFO`` will be answered using the contents of the
    designated file.  (This provider is used to wrap ``.egg-info`` files
    installed by vendor-supplied system packages.)


Utility Functions
=================

In addition to its high-level APIs, ``pkg_resources`` also includes several
generally-useful utility routines.  These routines are used to implement the
high-level APIs, but can also be quite useful by themselves.


Parsing Utilities
-----------------

``parse_version(version)``
    Parsed a project's version string as defined by PEP 440. The returned
    value will be an object that represents the version. These objects may
    be compared to each other and sorted. The sorting algorithm is as defined
    by PEP 440 with the addition that any version which is not a valid PEP 440
    version will be considered less than any valid PEP 440 version and the
    invalid versions will continue sorting using the original algorithm.

.. _yield_lines():

``yield_lines(strs)``
    Yield non-empty/non-comment lines from a string/unicode or a possibly-
    nested sequence thereof.  If `strs` is an instance of ``basestring``, it
    is split into lines, and each non-blank, non-comment line is yielded after
    stripping leading and trailing whitespace.  (Lines whose first non-blank
    character is ``#`` are considered comment lines.)

    If `strs` is not an instance of ``basestring``, it is iterated over, and
    each item is passed recursively to ``yield_lines()``, so that an arbitrarily
    nested sequence of strings, or sequences of sequences of strings can be
    flattened out to the lines contained therein.  So for example, passing
    a file object or a list of strings to ``yield_lines`` will both work.
    (Note that between each string in a sequence of strings there is assumed to
    be an implicit line break, so lines cannot bridge two strings in a
    sequence.)

    This routine is used extensively by ``pkg_resources`` to parse metadata
    and file formats of various kinds, and most other ``pkg_resources``
    parsing functions that yield multiple values will use it to break up their
    input.  However, this routine is idempotent, so calling ``yield_lines()``
    on the output of another call to ``yield_lines()`` is completely harmless.

``split_sections(strs)``
    Split a string (or possibly-nested iterable thereof), yielding ``(section,
    content)`` pairs found using an ``.ini``-like syntax.  Each ``section`` is
    a whitespace-stripped version of the section name ("``[section]``")
    and each ``content`` is a list of stripped lines excluding blank lines and
    comment-only lines.  If there are any non-blank, non-comment lines before
    the first section header, they're yielded in a first ``section`` of
    ``None``.

    This routine uses ``yield_lines()`` as its front end, so you can pass in
    anything that ``yield_lines()`` accepts, such as an open text file, string,
    or sequence of strings.  ``ValueError`` is raised if a malformed section
    header is found (i.e. a line starting with ``[`` but not ending with
    ``]``).

    Note that this simplistic parser assumes that any line whose first nonblank
    character is ``[`` is a section heading, so it can't support .ini format
    variations that allow ``[`` as the first nonblank character on other lines.

``safe_name(name)``
    Return a "safe" form of a project's name, suitable for use in a
    ``Requirement`` string, as a distribution name, or a PyPI project name.
    All non-alphanumeric runs are condensed to single "-" characters, such that
    a name like "The $$$ Tree" becomes "The-Tree".  Note that if you are
    generating a filename from this value you should combine it with a call to
    ``to_filename()`` so all dashes ("-") are replaced by underscores ("_").
    See ``to_filename()``.

``safe_version(version)``
    This will return the normalized form of any PEP 440 version, if the version
    string is not PEP 440 compatible than it is similar to ``safe_name()``
    except that spaces in the input become dots, and dots are allowed to exist
    in the output.  As with ``safe_name()``, if you are generating a filename
    from this you should replace any "-" characters in the output with
    underscores.

``safe_extra(extra)``
    Return a "safe" form of an extra's name, suitable for use in a requirement
    string or a setup script's ``extras_require`` keyword.  This routine is
    similar to ``safe_name()`` except that non-alphanumeric runs are replaced
    by a single underbar (``_``), and the result is lowercased.

``to_filename(name_or_version)``
    Escape a name or version string so it can be used in a dash-separated
    filename (or ``#egg=name-version`` tag) without ambiguity.  You
    should only pass in values that were returned by ``safe_name()`` or
    ``safe_version()``.


Platform Utilities
------------------

``get_build_platform()``
    Return this platform's identifier string.  For Windows, the return value
    is ``"win32"``, and for Mac OS X it is a string of the form
    ``"macosx-10.4-ppc"``.  All other platforms return the same uname-based
    string that the ``distutils.util.get_platform()`` function returns.
    This string is the minimum platform version required by distributions built
    on the local machine.  (Backward compatibility note: setuptools versions
    prior to 0.6b1 called this function ``get_platform()``, and the function is
    still available under that name for backward compatibility reasons.)

``get_supported_platform()`` (New in 0.6b1)
    This is the similar to ``get_build_platform()``, but is the maximum
    platform version that the local machine supports.  You will usually want
    to use this value as the ``provided`` argument to the
    ``compatible_platforms()`` function.

``compatible_platforms(provided, required)``
    Return true if a distribution built on the `provided` platform may be used
    on the `required` platform.  If either platform value is ``None``, it is
    considered a wildcard, and the platforms are therefore compatible.
    Likewise, if the platform strings are equal, they're also considered
    compatible, and ``True`` is returned.  Currently, the only non-equal
    platform strings that are considered compatible are Mac OS X platform
    strings with the same hardware type (e.g. ``ppc``) and major version
    (e.g. ``10``) with the `provided` platform's minor version being less than
    or equal to the `required` platform's minor version.

``get_default_cache()``
    Determine the default cache location for extracting resources from zipped
    eggs.  This routine returns the ``PYTHON_EGG_CACHE`` environment variable,
    if set.  Otherwise, on Windows, it returns a "Python-Eggs" subdirectory of
    the user's "Application Data" directory.  On all other systems, it returns
    ``os.path.expanduser("~/.python-eggs")`` if ``PYTHON_EGG_CACHE`` is not
    set.


PEP 302 Utilities
-----------------

``get_importer(path_item)``
    Retrieve a PEP 302 "importer" for the given path item (which need not
    actually be on ``sys.path``).  This routine simulates the PEP 302 protocol
    for obtaining an "importer" object.  It first checks for an importer for
    the path item in ``sys.path_importer_cache``, and if not found it calls
    each of the ``sys.path_hooks`` and caches the result if a good importer is
    found.  If no importer is found, this routine returns an ``ImpWrapper``
    instance that wraps the builtin import machinery as a PEP 302-compliant
    "importer" object.  This ``ImpWrapper`` is *not* cached; instead a new
    instance is returned each time.

    (Note: When run under Python 2.5, this function is simply an alias for
    ``pkgutil.get_importer()``, and instead of ``pkg_resources.ImpWrapper``
    instances, it may return ``pkgutil.ImpImporter`` instances.)


File/Path Utilities
-------------------

``ensure_directory(path)``
    Ensure that the parent directory (``os.path.dirname``) of `path` actually
    exists, using ``os.makedirs()`` if necessary.

``normalize_path(path)``
    Return a "normalized" version of `path`, such that two paths represent
    the same filesystem location if they have equal ``normalized_path()``
    values.  Specifically, this is a shortcut for calling ``os.path.realpath``
    and ``os.path.normcase`` on `path`.  Unfortunately, on certain platforms
    (notably Cygwin and Mac OS X) the ``normcase`` function does not accurately
    reflect the platform's case-sensitivity, so there is always the possibility
    of two apparently-different paths being equal on such platforms.

History
-------

0.6c9
 * Fix ``resource_listdir('')`` always returning an empty list for zipped eggs.

0.6c7
 * Fix package precedence problem where single-version eggs installed in
   ``site-packages`` would take precedence over ``.egg`` files (or directories)
   installed in ``site-packages``.

0.6c6
 * Fix extracted C extensions not having executable permissions under Cygwin.

 * Allow ``.egg-link`` files to contain relative paths.

 * Fix cache dir defaults on Windows when multiple environment vars are needed
   to construct a path.

0.6c4
 * Fix "dev" versions being considered newer than release candidates.

0.6c3
 * Python 2.5 compatibility fixes.

0.6c2
 * Fix a problem with eggs specified directly on ``PYTHONPATH`` on
   case-insensitive filesystems possibly not showing up in the default
   working set, due to differing normalizations of ``sys.path`` entries.

0.6b3
 * Fixed a duplicate path insertion problem on case-insensitive filesystems.

0.6b1
 * Split ``get_platform()`` into ``get_supported_platform()`` and
   ``get_build_platform()`` to work around a Mac versioning problem that caused
   the behavior of ``compatible_platforms()`` to be platform specific.

 * Fix entry point parsing when a standalone module name has whitespace
   between it and the extras.

0.6a11
 * Added ``ExtractionError`` and ``ResourceManager.extraction_error()`` so that
   cache permission problems get a more user-friendly explanation of the
   problem, and so that programs can catch and handle extraction errors if they
   need to.

0.6a10
 * Added the ``extras`` attribute to ``Distribution``, the ``find_plugins()``
   method to ``WorkingSet``, and the ``__add__()`` and ``__iadd__()`` methods
   to ``Environment``.

 * ``safe_name()`` now allows dots in project names.

 * There is a new ``to_filename()`` function that escapes project names and
   versions for safe use in constructing egg filenames from a Distribution
   object's metadata.

 * Added ``Distribution.clone()`` method, and keyword argument support to other
   ``Distribution`` constructors.

 * Added the ``DEVELOP_DIST`` precedence, and automatically assign it to
   eggs using ``.egg-info`` format.

0.6a9
 * Don't raise an error when an invalid (unfinished) distribution is found
   unless absolutely necessary.  Warn about skipping invalid/unfinished eggs
   when building an Environment.

 * Added support for ``.egg-info`` files or directories with version/platform
   information embedded in the filename, so that system packagers have the
   option of including ``PKG-INFO`` files to indicate the presence of a
   system-installed egg, without needing to use ``.egg`` directories, zipfiles,
   or ``.pth`` manipulation.

 * Changed ``parse_version()`` to remove dashes before pre-release tags, so
   that ``0.2-rc1`` is considered an *older* version than ``0.2``, and is equal
   to ``0.2rc1``.  The idea that a dash *always* meant a post-release version
   was highly non-intuitive to setuptools users and Python developers, who
   seem to want to use ``-rc`` version numbers a lot.

0.6a8
 * Fixed a problem with ``WorkingSet.resolve()`` that prevented version
   conflicts from being detected at runtime.

 * Improved runtime conflict warning message to identify a line in the user's
   program, rather than flagging the ``warn()`` call in ``pkg_resources``.

 * Avoid giving runtime conflict warnings for namespace packages, even if they
   were declared by a different package than the one currently being activated.

 * Fix path insertion algorithm for case-insensitive filesystems.

 * Fixed a problem with nested namespace packages (e.g. ``peak.util``) not
   being set as an attribute of their parent package.

0.6a6
 * Activated distributions are now inserted in ``sys.path`` (and the working
   set) just before the directory that contains them, instead of at the end.
   This allows e.g. eggs in ``site-packages`` to override unmanaged modules in
   the same location, and allows eggs found earlier on ``sys.path`` to override
   ones found later.

 * When a distribution is activated, it now checks whether any contained
   non-namespace modules have already been imported and issues a warning if
   a conflicting module has already been imported.

 * Changed dependency processing so that it's breadth-first, allowing a
   depender's preferences to override those of a dependee, to prevent conflicts
   when a lower version is acceptable to the dependee, but not the depender.

 * Fixed a problem extracting zipped files on Windows, when the egg in question
   has had changed contents but still has the same version number.

0.6a4
 * Fix a bug in ``WorkingSet.resolve()`` that was introduced in 0.6a3.

0.6a3
 * Added ``safe_extra()`` parsing utility routine, and use it for Requirement,
   EntryPoint, and Distribution objects' extras handling.

0.6a1
 * Enhanced performance of ``require()`` and related operations when all
   requirements are already in the working set, and enhanced performance of
   directory scanning for distributions.

 * Fixed some problems using ``pkg_resources`` w/PEP 302 loaders other than
   ``zipimport``, and the previously-broken "eager resource" support.

 * Fixed ``pkg_resources.resource_exists()`` not working correctly, along with
   some other resource API bugs.

 * Many API changes and enhancements:

   * Added ``EntryPoint``, ``get_entry_map``, ``load_entry_point``, and
     ``get_entry_info`` APIs for dynamic plugin discovery.

   * ``list_resources`` is now ``resource_listdir`` (and it actually works)

   * Resource API functions like ``resource_string()`` that accepted a package
     name and resource name, will now also accept a ``Requirement`` object in
     place of the package name (to allow access to non-package data files in
     an egg).

   * ``get_provider()`` will now accept a ``Requirement`` instance or a module
     name.  If it is given a ``Requirement``, it will return a corresponding
     ``Distribution`` (by calling ``require()`` if a suitable distribution
     isn't already in the working set), rather than returning a metadata and
     resource provider for a specific module.  (The difference is in how
     resource paths are interpreted; supplying a module name means resources
     path will be module-relative, rather than relative to the distribution's
     root.)

   * ``Distribution`` objects now implement the ``IResourceProvider`` and
     ``IMetadataProvider`` interfaces, so you don't need to reference the (no
     longer available) ``metadata`` attribute to get at these interfaces.

   * ``Distribution`` and ``Requirement`` both have a ``project_name``
     attribute for the project name they refer to.  (Previously these were
     ``name`` and ``distname`` attributes.)

   * The ``path`` attribute of ``Distribution`` objects is now ``location``,
     because it isn't necessarily a filesystem path (and hasn't been for some
     time now).  The ``location`` of ``Distribution`` objects in the filesystem
     should always be normalized using ``pkg_resources.normalize_path()``; all
     of the setuptools and EasyInstall code that generates distributions from
     the filesystem (including ``Distribution.from_filename()``) ensure this
     invariant, but if you use a more generic API like ``Distribution()`` or
     ``Distribution.from_location()`` you should take care that you don't
     create a distribution with an un-normalized filesystem path.

   * ``Distribution`` objects now have an ``as_requirement()`` method that
     returns a ``Requirement`` for the distribution's project name and version.

   * Distribution objects no longer have an ``installed_on()`` method, and the
     ``install_on()`` method is now ``activate()`` (but may go away altogether
     soon).  The ``depends()`` method has also been renamed to ``requires()``,
     and ``InvalidOption`` is now ``UnknownExtra``.

   * ``find_distributions()`` now takes an additional argument called ``only``,
     that tells it to only yield distributions whose location is the passed-in
     path.  (It defaults to False, so that the default behavior is unchanged.)

   * ``AvailableDistributions`` is now called ``Environment``, and the
     ``get()``, ``__len__()``, and ``__contains__()`` methods were removed,
     because they weren't particularly useful.  ``__getitem__()`` no longer
     raises ``KeyError``; it just returns an empty list if there are no
     distributions for the named project.

   * The ``resolve()`` method of ``Environment`` is now a method of
     ``WorkingSet`` instead, and the ``best_match()`` method now uses a working
     set instead of a path list as its second argument.

   * There is a new ``pkg_resources.add_activation_listener()`` API that lets
     you register a callback for notifications about distributions added to
     ``sys.path`` (including the distributions already on it).  This is
     basically a hook for extensible applications and frameworks to be able to
     search for plugin metadata in distributions added at runtime.

0.5a13
 * Fixed a bug in resource extraction from nested packages in a zipped egg.

0.5a12
 * Updated extraction/cache mechanism for zipped resources to avoid inter-
   process and inter-thread races during extraction.  The default cache
   location can now be set via the ``PYTHON_EGGS_CACHE`` environment variable,
   and the default Windows cache is now a ``Python-Eggs`` subdirectory of the
   current user's "Application Data" directory, if the ``PYTHON_EGGS_CACHE``
   variable isn't set.

0.5a10
 * Fix a problem with ``pkg_resources`` being confused by non-existent eggs on
   ``sys.path`` (e.g. if a user deletes an egg without removing it from the
   ``easy-install.pth`` file).

 * Fix a problem with "basket" support in ``pkg_resources``, where egg-finding
   never actually went inside ``.egg`` files.

 * Made ``pkg_resources`` import the module you request resources from, if it's
   not already imported.

0.5a4
 * ``pkg_resources.AvailableDistributions.resolve()`` and related methods now
   accept an ``installer`` argument: a callable taking one argument, a
   ``Requirement`` instance.  The callable must return a ``Distribution``
   object, or ``None`` if no distribution is found.  This feature is used by
   EasyInstall to resolve dependencies by recursively invoking itself.

0.4a4
 * Fix problems with ``resource_listdir()``, ``resource_isdir()`` and resource
   directory extraction for zipped eggs.

0.4a3
 * Fixed scripts not being able to see a ``__file__`` variable in ``__main__``

 * Fixed a problem with ``resource_isdir()`` implementation that was introduced
   in 0.4a2.

0.4a1
 * Fixed a bug in requirements processing for exact versions (i.e. ``==`` and
   ``!=``) when only one condition was included.

 * Added ``safe_name()`` and ``safe_version()`` APIs to clean up handling of
   arbitrary distribution names and versions found on PyPI.

0.3a4
 * ``pkg_resources`` now supports resource directories, not just the resources
   in them.  In particular, there are ``resource_listdir()`` and
   ``resource_isdir()`` APIs.

 * ``pkg_resources`` now supports "egg baskets" -- .egg zipfiles which contain
   multiple distributions in subdirectories whose names end with ``.egg``.
   Having such a "basket" in a directory on ``sys.path`` is equivalent to
   having the individual eggs in that directory, but the contained eggs can
   be individually added (or not) to ``sys.path``.  Currently, however, there
   is no automated way to create baskets.

 * Namespace package manipulation is now protected by the Python import lock.

0.3a1
 * Initial release.

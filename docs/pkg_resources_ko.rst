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
    개별 디스트리뷰션에 의해 선전되는 엔트리 포인트 사이에서는 특별한 순서가 존재하지 않는다.

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

    The project name is the only required portion of a requirement string, and
    if it's the only thing supplied, the requirement will accept any version
    of that project.

    The "extras" in a requirement are used to request optional features of a
    project, that may require additional project distributions in order to
    function.  For example, if the hypothetical "Report-O-Rama" project offered
    optional PDF support, it might require an additional library in order to
    provide that support.  Thus, a project needing Report-O-Rama's PDF features
    could use a requirement of ``Report-O-Rama[PDF]`` to request installation
    or activation of both Report-O-Rama and any libraries it needs in order to
    provide PDF support.  For example, you could use::

        easy_install.py Report-O-Rama[PDF]

    To install the necessary packages using the EasyInstall program, or call
    ``pkg_resources.require('Report-O-Rama[PDF]')`` to add the necessary
    distributions to sys.path at runtime.

    The "markers" in a requirement are used to specify when a requirement
    should be installed -- the requirement will be installed if the marker
    evaluates as true in the current environment. For example, specifying
    ``argparse;python_version<"2.7"`` will not install in an Python 2.7 or 3.3
    environment, but will in a Python 2.6 environment.

``Requirement`` Methods and Attributes
--------------------------------------

``__contains__(dist_or_version)``
    Return true if `dist_or_version` fits the criteria for this requirement.
    If `dist_or_version` is a ``Distribution`` object, its project name must
    match the requirement's project name, and its version must meet the
    requirement's version criteria.  If `dist_or_version` is a string, it is
    parsed using the ``parse_version()`` utility function.  Otherwise, it is
    assumed to be an already-parsed version.

    The ``Requirement`` object's version specifiers (``.specs``) are internally
    sorted into ascending version order, and used to establish what ranges of
    versions are acceptable.  Adjacent redundant conditions are effectively
    consolidated (e.g. ``">1, >2"`` produces the same results as ``">2"``, and
    ``"<2,<3"`` produces the same results as``"<2"``). ``"!="`` versions are
    excised from the ranges they fall within.  The version being tested for
    acceptability is then checked for membership in the resulting ranges.

``__eq__(other_requirement)``
    A requirement compares equal to another requirement if they have
    case-insensitively equal project names, version specifiers, and "extras".
    (The order that extras and version specifiers are in is also ignored.)
    Equal requirements also have equal hashes, so that requirements can be
    used in sets or as dictionary keys.

``__str__()``
    The string form of a ``Requirement`` is a string that, if passed to
    ``Requirement.parse()``, would return an equal ``Requirement`` object.

``project_name``
    The name of the required project

``key``
    An all-lowercase version of the ``project_name``, useful for comparison
    or indexing.

``extras``
    A tuple of names of "extras" that this requirement calls for.  (These will
    be all-lowercase and normalized using the ``safe_extra()`` parsing utility
    function, so they may not exactly equal the extras the requirement was
    created with.)

``specs``
    A list of ``(op,version)`` tuples, sorted in ascending parsed-version
    order.  The `op` in each tuple is a comparison operator, represented as
    a string.  The `version` is the (unparsed) version number.

``marker``
    An instance of ``packaging.markers.Marker`` that allows evaluation
    against the current environment. May be None if no marker specified.

``url``
    The location to download the requirement from if specified.

Entry Points
============

Entry points are a simple way for distributions to "advertise" Python objects
(such as functions or classes) for use by other distributions.  Extensible
applications and frameworks can search for entry points with a particular name
or group, either from a specific distribution or from all active distributions
on sys.path, and then inspect or load the advertised objects at will.

Entry points belong to "groups" which are named with a dotted name similar to
a Python package or module name.  For example, the ``setuptools`` package uses
an entry point named ``distutils.commands`` in order to find commands defined
by distutils extensions.  ``setuptools`` treats the names of entry points
defined in that group as the acceptable commands for a setup script.

In a similar way, other packages can define their own entry point groups,
either using dynamic names within the group (like ``distutils.commands``), or
possibly using predefined names within the group.  For example, a blogging
framework that offers various pre- or post-publishing hooks might define an
entry point group and look for entry points named "pre_process" and
"post_process" within that group.

To advertise an entry point, a project needs to use ``setuptools`` and provide
an ``entry_points`` argument to ``setup()`` in its setup script, so that the
entry points will be included in the distribution's metadata.  For more
details, see the ``setuptools`` documentation.  (XXX link here to setuptools)

Each project distribution can advertise at most one entry point of a given
name within the same entry point group.  For example, a distutils extension
could advertise two different ``distutils.commands`` entry points, as long as
they had different names.  However, there is nothing that prevents *different*
projects from advertising entry points of the same name in the same group.  In
some cases, this is a desirable thing, since the application or framework that
uses the entry points may be calling them as hooks, or in some other way
combining them.  It is up to the application or framework to decide what to do
if multiple distributions advertise an entry point; some possibilities include
using both entry points, displaying an error message, using the first one found
in sys.path order, etc.


Convenience API
---------------

In the following functions, the `dist` argument can be a ``Distribution``
instance, a ``Requirement`` instance, or a string specifying a requirement
(i.e. project name, version, etc.).  If the argument is a string or
``Requirement``, the specified distribution is located (and added to sys.path
if not already present).  An error will be raised if a matching distribution is
not available.

The `group` argument should be a string containing a dotted identifier,
identifying an entry point group.  If you are defining an entry point group,
you should include some portion of your package's name in the group name so as
to avoid collision with other packages' entry point groups.

``load_entry_point(dist, group, name)``
    Load the named entry point from the specified distribution, or raise
    ``ImportError``.

``get_entry_info(dist, group, name)``
    Return an ``EntryPoint`` object for the given `group` and `name` from
    the specified distribution.  Returns ``None`` if the distribution has not
    advertised a matching entry point.

``get_entry_map(dist, group=None)``
    Return the distribution's entry point map for `group`, or the full entry
    map for the distribution.  This function always returns a dictionary,
    even if the distribution advertises no entry points.  If `group` is given,
    the dictionary maps entry point names to the corresponding ``EntryPoint``
    object.  If `group` is None, the dictionary maps group names to
    dictionaries that then map entry point names to the corresponding
    ``EntryPoint`` instance in that group.

``iter_entry_points(group, name=None)``
    Yield entry point objects from `group` matching `name`.

    If `name` is None, yields all entry points in `group` from all
    distributions in the working set on sys.path, otherwise only ones matching
    both `group` and `name` are yielded.  Entry points are yielded from
    the active distributions in the order that the distributions appear on
    sys.path.  (Within entry points for a particular distribution, however,
    there is no particular ordering.)

    (This API is actually a method of the global ``working_set`` object; see
    the section above on `Basic WorkingSet Methods`_ for more information.)


Creating and Parsing
--------------------

``EntryPoint(name, module_name, attrs=(), extras=(), dist=None)``
    Create an ``EntryPoint`` instance.  `name` is the entry point name.  The
    `module_name` is the (dotted) name of the module containing the advertised
    object.  `attrs` is an optional tuple of names to look up from the
    module to obtain the advertised object.  For example, an `attrs` of
    ``("foo","bar")`` and a `module_name` of ``"baz"`` would mean that the
    advertised object could be obtained by the following code::

        import baz
        advertised_object = baz.foo.bar

    The `extras` are an optional tuple of "extra feature" names that the
    distribution needs in order to provide this entry point.  When the
    entry point is loaded, these extra features are looked up in the `dist`
    argument to find out what other distributions may need to be activated
    on sys.path; see the ``load()`` method for more details.  The `extras`
    argument is only meaningful if `dist` is specified.  `dist` must be
    a ``Distribution`` instance.

``EntryPoint.parse(src, dist=None)`` (classmethod)
    Parse a single entry point from string `src`

    Entry point syntax follows the form::

        name = some.module:some.attr [extra1,extra2]

    The entry name and module name are required, but the ``:attrs`` and
    ``[extras]`` parts are optional, as is the whitespace shown between
    some of the items.  The `dist` argument is passed through to the
    ``EntryPoint()`` constructor, along with the other values parsed from
    `src`.

``EntryPoint.parse_group(group, lines, dist=None)`` (classmethod)
    Parse `lines` (a string or sequence of lines) to create a dictionary
    mapping entry point names to ``EntryPoint`` objects.  ``ValueError`` is
    raised if entry point names are duplicated, if `group` is not a valid
    entry point group name, or if there are any syntax errors.  (Note: the
    `group` parameter is used only for validation and to create more
    informative error messages.)  If `dist` is provided, it will be used to
    set the ``dist`` attribute of the created ``EntryPoint`` objects.

``EntryPoint.parse_map(data, dist=None)`` (classmethod)
    Parse `data` into a dictionary mapping group names to dictionaries mapping
    entry point names to ``EntryPoint`` objects.  If `data` is a dictionary,
    then the keys are used as group names and the values are passed to
    ``parse_group()`` as the `lines` argument.  If `data` is a string or
    sequence of lines, it is first split into .ini-style sections (using
    the ``split_sections()`` utility function) and the section names are used
    as group names.  In either case, the `dist` argument is passed through to
    ``parse_group()`` so that the entry points will be linked to the specified
    distribution.


``EntryPoint`` Objects
----------------------

For simple introspection, ``EntryPoint`` objects have attributes that
correspond exactly to the constructor argument names: ``name``,
``module_name``, ``attrs``, ``extras``, and ``dist`` are all available.  In
addition, the following methods are provided:

``load()``
    Load the entry point, returning the advertised Python object.  Effectively
    calls ``self.require()`` then returns ``self.resolve()``.

``require(env=None, installer=None)``
    Ensure that any "extras" needed by the entry point are available on
    sys.path.  ``UnknownExtra`` is raised if the ``EntryPoint`` has ``extras``,
    but no ``dist``, or if the named extras are not defined by the
    distribution.  If `env` is supplied, it must be an ``Environment``, and it
    will be used to search for needed distributions if they are not already
    present on sys.path.  If `installer` is supplied, it must be a callable
    taking a ``Requirement`` instance and returning a matching importable
    ``Distribution`` instance or None.

``resolve()``
    Resolve the entry point from its module and attrs, returning the advertised
    Python object. Raises ``ImportError`` if it cannot be obtained.

``__str__()``
    The string form of an ``EntryPoint`` is a string that could be passed to
    ``EntryPoint.parse()`` to produce an equivalent ``EntryPoint``.


``Distribution`` Objects
========================

``Distribution`` objects represent collections of Python code that may or may
not be importable, and may or may not have metadata and resources associated
with them.  Their metadata may include information such as what other projects
the distribution depends on, what entry points the distribution advertises, and
so on.


Getting or Creating Distributions
---------------------------------

Most commonly, you'll obtain ``Distribution`` objects from a ``WorkingSet`` or
an ``Environment``.  (See the sections above on `WorkingSet Objects`_ and
`Environment Objects`_, which are containers for active distributions and
available distributions, respectively.)  You can also obtain ``Distribution``
objects from one of these high-level APIs:

``find_distributions(path_item, only=False)``
    Yield distributions accessible via `path_item`.  If `only` is true, yield
    only distributions whose ``location`` is equal to `path_item`.  In other
    words, if `only` is true, this yields any distributions that would be
    importable if `path_item` were on ``sys.path``.  If `only` is false, this
    also yields distributions that are "in" or "under" `path_item`, but would
    not be importable unless their locations were also added to ``sys.path``.

``get_distribution(dist_spec)``
    Return a ``Distribution`` object for a given ``Requirement`` or string.
    If `dist_spec` is already a ``Distribution`` instance, it is returned.
    If it is a ``Requirement`` object or a string that can be parsed into one,
    it is used to locate and activate a matching distribution, which is then
    returned.

However, if you're creating specialized tools for working with distributions,
or creating a new distribution format, you may also need to create
``Distribution`` objects directly, using one of the three constructors below.

These constructors all take an optional `metadata` argument, which is used to
access any resources or metadata associated with the distribution.  `metadata`
must be an object that implements the ``IResourceProvider`` interface, or None.
If it is None, an ``EmptyProvider`` is used instead.  ``Distribution`` objects
implement both the `IResourceProvider`_ and `IMetadataProvider Methods`_ by
delegating them to the `metadata` object.

``Distribution.from_location(location, basename, metadata=None, **kw)`` (classmethod)
    Create a distribution for `location`, which must be a string such as a
    URL, filename, or other string that might be used on ``sys.path``.
    `basename` is a string naming the distribution, like ``Foo-1.2-py2.4.egg``.
    If `basename` ends with ``.egg``, then the project's name, version, python
    version and platform are extracted from the filename and used to set those
    properties of the created distribution.  Any additional keyword arguments
    are forwarded to the ``Distribution()`` constructor.

``Distribution.from_filename(filename, metadata=None**kw)`` (classmethod)
    Create a distribution by parsing a local filename.  This is a shorter way
    of saying  ``Distribution.from_location(normalize_path(filename),
    os.path.basename(filename), metadata)``.  In other words, it creates a
    distribution whose location is the normalize form of the filename, parsing
    name and version information from the base portion of the filename.  Any
    additional keyword arguments are forwarded to the ``Distribution()``
    constructor.

``Distribution(location,metadata,project_name,version,py_version,platform,precedence)``
    Create a distribution by setting its properties.  All arguments are
    optional and default to None, except for `py_version` (which defaults to
    the current Python version) and `precedence` (which defaults to
    ``EGG_DIST``; for more details see ``precedence`` under `Distribution
    Attributes`_ below).  Note that it's usually easier to use the
    ``from_filename()`` or ``from_location()`` constructors than to specify
    all these arguments individually.


``Distribution`` Attributes
---------------------------

location
    A string indicating the distribution's location.  For an importable
    distribution, this is the string that would be added to ``sys.path`` to
    make it actively importable.  For non-importable distributions, this is
    simply a filename, URL, or other way of locating the distribution.

project_name
    A string, naming the project that this distribution is for.  Project names
    are defined by a project's setup script, and they are used to identify
    projects on PyPI.  When a ``Distribution`` is constructed, the
    `project_name` argument is passed through the ``safe_name()`` utility
    function to filter out any unacceptable characters.

key
    ``dist.key`` is short for ``dist.project_name.lower()``.  It's used for
    case-insensitive comparison and indexing of distributions by project name.

extras
    A list of strings, giving the names of extra features defined by the
    project's dependency list (the ``extras_require`` argument specified in
    the project's setup script).

version
    A string denoting what release of the project this distribution contains.
    When a ``Distribution`` is constructed, the `version` argument is passed
    through the ``safe_version()`` utility function to filter out any
    unacceptable characters.  If no `version` is specified at construction
    time, then attempting to access this attribute later will cause the
    ``Distribution`` to try to discover its version by reading its ``PKG-INFO``
    metadata file.  If ``PKG-INFO`` is unavailable or can't be parsed,
    ``ValueError`` is raised.

parsed_version
    The ``parsed_version`` is an object representing a "parsed" form of the
    distribution's ``version``.  ``dist.parsed_version`` is a shortcut for
    calling ``parse_version(dist.version)``.  It is used to compare or sort
    distributions by version.  (See the `Parsing Utilities`_ section below for
    more information on the ``parse_version()`` function.)  Note that accessing
    ``parsed_version`` may result in a ``ValueError`` if the ``Distribution``
    was constructed without a `version` and without `metadata` capable of
    supplying the missing version info.

py_version
    The major/minor Python version the distribution supports, as a string.
    For example, "2.7" or "3.4".  The default is the current version of Python.

platform
    A string representing the platform the distribution is intended for, or
    ``None`` if the distribution is "pure Python" and therefore cross-platform.
    See `Platform Utilities`_ below for more information on platform strings.

precedence
    A distribution's ``precedence`` is used to determine the relative order of
    two distributions that have the same ``project_name`` and
    ``parsed_version``.  The default precedence is ``pkg_resources.EGG_DIST``,
    which is the highest (i.e. most preferred) precedence.  The full list
    of predefined precedences, from most preferred to least preferred, is:
    ``EGG_DIST``, ``BINARY_DIST``, ``SOURCE_DIST``, ``CHECKOUT_DIST``, and
    ``DEVELOP_DIST``.  Normally, precedences other than ``EGG_DIST`` are used
    only by the ``setuptools.package_index`` module, when sorting distributions
    found in a package index to determine their suitability for installation.
    "System" and "Development" eggs (i.e., ones that use the ``.egg-info``
    format), however, are automatically given a precedence of ``DEVELOP_DIST``.



``Distribution`` Methods
------------------------

``activate(path=None)``
    Ensure distribution is importable on `path`.  If `path` is None,
    ``sys.path`` is used instead.  This ensures that the distribution's
    ``location`` is in the `path` list, and it also performs any necessary
    namespace package fixups or declarations.  (That is, if the distribution
    contains namespace packages, this method ensures that they are declared,
    and that the distribution's contents for those namespace packages are
    merged with the contents provided by any other active distributions.  See
    the section above on `Namespace Package Support`_ for more information.)

    ``pkg_resources`` adds a notification callback to the global ``working_set``
    that ensures this method is called whenever a distribution is added to it.
    Therefore, you should not normally need to explicitly call this method.
    (Note that this means that namespace packages on ``sys.path`` are always
    imported as soon as ``pkg_resources`` is, which is another reason why
    namespace packages should not contain any code or import statements.)

``as_requirement()``
    Return a ``Requirement`` instance that matches this distribution's project
    name and version.

``requires(extras=())``
    List the ``Requirement`` objects that specify this distribution's
    dependencies.  If `extras` is specified, it should be a sequence of names
    of "extras" defined by the distribution, and the list returned will then
    include any dependencies needed to support the named "extras".

``clone(**kw)``
    Create a copy of the distribution.  Any supplied keyword arguments override
    the corresponding argument to the ``Distribution()`` constructor, allowing
    you to change some of the copied distribution's attributes.

``egg_name()``
    Return what this distribution's standard filename should be, not including
    the ".egg" extension.  For example, a distribution for project "Foo"
    version 1.2 that runs on Python 2.3 for Windows would have an ``egg_name()``
    of ``Foo-1.2-py2.3-win32``.  Any dashes in the name or version are
    converted to underscores.  (``Distribution.from_location()`` will convert
    them back when parsing a ".egg" file name.)

``__cmp__(other)``, ``__hash__()``
    Distribution objects are hashed and compared on the basis of their parsed
    version and precedence, followed by their key (lowercase project name),
    location, Python version, and platform.

The following methods are used to access ``EntryPoint`` objects advertised
by the distribution.  See the section above on `Entry Points`_ for more
detailed information about these operations:

``get_entry_info(group, name)``
    Return the ``EntryPoint`` object for `group` and `name`, or None if no
    such point is advertised by this distribution.

``get_entry_map(group=None)``
    Return the entry point map for `group`.  If `group` is None, return
    a dictionary mapping group names to entry point maps for all groups.
    (An entry point map is a dictionary of entry point names to ``EntryPoint``
    objects.)

``load_entry_point(group, name)``
    Short for ``get_entry_info(group, name).load()``.  Returns the object
    advertised by the named entry point, or raises ``ImportError`` if
    the entry point isn't advertised by this distribution, or there is some
    other import problem.

In addition to the above methods, ``Distribution`` objects also implement all
of the `IResourceProvider`_ and `IMetadataProvider Methods`_ (which are
documented in later sections):

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

If the distribution was created with a `metadata` argument, these resource and
metadata access methods are all delegated to that `metadata` provider.
Otherwise, they are delegated to an ``EmptyProvider``, so that the distribution
will appear to have no resources or metadata.  This delegation approach is used
so that supporting custom importers or new distribution formats can be done
simply by creating an appropriate `IResourceProvider`_ implementation; see the
section below on `Supporting Custom Importers`_ for more details.


``ResourceManager`` API
=======================

The ``ResourceManager`` class provides uniform access to package resources,
whether those resources exist as files and directories or are compressed in
an archive of some kind.

Normally, you do not need to create or explicitly manage ``ResourceManager``
instances, as the ``pkg_resources`` module creates a global instance for you,
and makes most of its methods available as top-level names in the
``pkg_resources`` module namespace.  So, for example, this code actually
calls the ``resource_string()`` method of the global ``ResourceManager``::

    import pkg_resources
    my_data = pkg_resources.resource_string(__name__, "foo.dat")

Thus, you can use the APIs below without needing an explicit
``ResourceManager`` instance; just import and use them as needed.


Basic Resource Access
---------------------

In the following methods, the `package_or_requirement` argument may be either
a Python package/module name (e.g. ``foo.bar``) or a ``Requirement`` instance.
If it is a package or module name, the named module or package must be
importable (i.e., be in a distribution or directory on ``sys.path``), and the
`resource_name` argument is interpreted relative to the named package.  (Note
that if a module name is used, then the resource name is relative to the
package immediately containing the named module.  Also, you should not use use
a namespace package name, because a namespace package can be spread across
multiple distributions, and is therefore ambiguous as to which distribution
should be searched for the resource.)

If it is a ``Requirement``, then the requirement is automatically resolved
(searching the current ``Environment`` if necessary) and a matching
distribution is added to the ``WorkingSet`` and ``sys.path`` if one was not
already present.  (Unless the ``Requirement`` can't be satisfied, in which
case an exception is raised.)  The `resource_name` argument is then interpreted
relative to the root of the identified distribution; i.e. its first path
segment will be treated as a peer of the top-level modules or packages in the
distribution.

Note that resource names must be ``/``-separated paths and cannot be absolute
(i.e. no leading ``/``) or contain relative names like ``".."``.  Do *not* use
``os.path`` routines to manipulate resource paths, as they are *not* filesystem
paths.

``resource_exists(package_or_requirement, resource_name)``
    Does the named resource exist?  Return ``True`` or ``False`` accordingly.

``resource_stream(package_or_requirement, resource_name)``
    Return a readable file-like object for the specified resource; it may be
    an actual file, a ``StringIO``, or some similar object.  The stream is
    in "binary mode", in the sense that whatever bytes are in the resource
    will be read as-is.

``resource_string(package_or_requirement, resource_name)``
    Return the specified resource as a string.  The resource is read in
    binary fashion, such that the returned string contains exactly the bytes
    that are stored in the resource.

``resource_isdir(package_or_requirement, resource_name)``
    Is the named resource a directory?  Return ``True`` or ``False``
    accordingly.

``resource_listdir(package_or_requirement, resource_name)``
    List the contents of the named resource directory, just like ``os.listdir``
    except that it works even if the resource is in a zipfile.

Note that only ``resource_exists()`` and ``resource_isdir()`` are insensitive
as to the resource type.  You cannot use ``resource_listdir()`` on a file
resource, and you can't use ``resource_string()`` or ``resource_stream()`` on
directory resources.  Using an inappropriate method for the resource type may
result in an exception or undefined behavior, depending on the platform and
distribution format involved.


Resource Extraction
-------------------

``resource_filename(package_or_requirement, resource_name)``
    Sometimes, it is not sufficient to access a resource in string or stream
    form, and a true filesystem filename is needed.  In such cases, you can
    use this method (or module-level function) to obtain a filename for a
    resource.  If the resource is in an archive distribution (such as a zipped
    egg), it will be extracted to a cache directory, and the filename within
    the cache will be returned.  If the named resource is a directory, then
    all resources within that directory (including subdirectories) are also
    extracted.  If the named resource is a C extension or "eager resource"
    (see the ``setuptools`` documentation for details), then all C extensions
    and eager resources are extracted at the same time.

    Archived resources are extracted to a cache location that can be managed by
    the following two methods:

``set_extraction_path(path)``
    Set the base path where resources will be extracted to, if needed.

    If you do not call this routine before any extractions take place, the
    path defaults to the return value of ``get_default_cache()``.  (Which is
    based on the ``PYTHON_EGG_CACHE`` environment variable, with various
    platform-specific fallbacks.  See that routine's documentation for more
    details.)

    Resources are extracted to subdirectories of this path based upon
    information given by the resource provider.  You may set this to a
    temporary directory, but then you must call ``cleanup_resources()`` to
    delete the extracted files when done.  There is no guarantee that
    ``cleanup_resources()`` will be able to remove all extracted files.  (On
    Windows, for example, you can't unlink .pyd or .dll files that are still
    in use.)

    Note that you may not change the extraction path for a given resource
    manager once resources have been extracted, unless you first call
    ``cleanup_resources()``.

``cleanup_resources(force=False)``
    Delete all extracted resource files and directories, returning a list
    of the file and directory names that could not be successfully removed.
    This function does not have any concurrency protection, so it should
    generally only be called when the extraction path is a temporary
    directory exclusive to a single process.  This method is not
    automatically called; you must call it explicitly or register it as an
    ``atexit`` function if you wish to ensure cleanup of a temporary
    directory used for extractions.


"Provider" Interface
--------------------

If you are implementing an ``IResourceProvider`` and/or ``IMetadataProvider``
for a new distribution archive format, you may need to use the following
``IResourceManager`` methods to co-ordinate extraction of resources to the
filesystem.  If you're not implementing an archive format, however, you have
no need to use these methods.  Unlike the other methods listed above, they are
*not* available as top-level functions tied to the global ``ResourceManager``;
you must therefore have an explicit ``ResourceManager`` instance to use them.

``get_cache_path(archive_name, names=())``
    Return absolute location in cache for `archive_name` and `names`

    The parent directory of the resulting path will be created if it does
    not already exist.  `archive_name` should be the base filename of the
    enclosing egg (which may not be the name of the enclosing zipfile!),
    including its ".egg" extension.  `names`, if provided, should be a
    sequence of path name parts "under" the egg's extraction location.

    This method should only be called by resource providers that need to
    obtain an extraction location, and only for names they intend to
    extract, as it tracks the generated names for possible cleanup later.

``extraction_error()``
    Raise an ``ExtractionError`` describing the active exception as interfering
    with the extraction process.  You should call this if you encounter any
    OS errors extracting the file to the cache path; it will format the
    operating system exception for you, and add other information to the
    ``ExtractionError`` instance that may be needed by programs that want to
    wrap or handle extraction errors themselves.

``postprocess(tempname, filename)``
    Perform any platform-specific postprocessing of `tempname`.
    Resource providers should call this method ONLY after successfully
    extracting a compressed resource.  They must NOT call it on resources
    that are already in the filesystem.

    `tempname` is the current (temporary) name of the file, and `filename`
    is the name it will be renamed to by the caller after this routine
    returns.


Metadata API
============

The metadata API is used to access metadata resources bundled in a pluggable
distribution.  Metadata resources are virtual files or directories containing
information about the distribution, such as might be used by an extensible
application or framework to connect "plugins".  Like other kinds of resources,
metadata resource names are ``/``-separated and should not contain ``..`` or
begin with a ``/``.  You should not use ``os.path`` routines to manipulate
resource paths.

The metadata API is provided by objects implementing the ``IMetadataProvider``
or ``IResourceProvider`` interfaces.  ``Distribution`` objects implement this
interface, as do objects returned by the ``get_provider()`` function:

``get_provider(package_or_requirement)``
    If a package name is supplied, return an ``IResourceProvider`` for the
    package.  If a ``Requirement`` is supplied, resolve it by returning a
    ``Distribution`` from the current working set (searching the current
    ``Environment`` if necessary and adding the newly found ``Distribution``
    to the working set).  If the named package can't be imported, or the
    ``Requirement`` can't be satisfied, an exception is raised.

    NOTE: if you use a package name rather than a ``Requirement``, the object
    you get back may not be a pluggable distribution, depending on the method
    by which the package was installed.  In particular, "development" packages
    and "single-version externally-managed" packages do not have any way to
    map from a package name to the corresponding project's metadata.  Do not
    write code that passes a package name to ``get_provider()`` and then tries
    to retrieve project metadata from the returned object.  It may appear to
    work when the named package is in an ``.egg`` file or directory, but
    it will fail in other installation scenarios.  If you want project
    metadata, you need to ask for a *project*, not a package.


``IMetadataProvider`` Methods
-----------------------------

The methods provided by objects (such as ``Distribution`` instances) that
implement the ``IMetadataProvider`` or ``IResourceProvider`` interfaces are:

``has_metadata(name)``
    Does the named metadata resource exist?

``metadata_isdir(name)``
    Is the named metadata resource a directory?

``metadata_listdir(name)``
    List of metadata names in the directory (like ``os.listdir()``)

``get_metadata(name)``
    Return the named metadata resource as a string.  The data is read in binary
    mode; i.e., the exact bytes of the resource file are returned.

``get_metadata_lines(name)``
    Yield named metadata resource as list of non-blank non-comment lines.  This
    is short for calling ``yield_lines(provider.get_metadata(name))``.  See the
    section on `yield_lines()`_ below for more information on the syntax it
    recognizes.

``run_script(script_name, namespace)``
    Execute the named script in the supplied namespace dictionary.  Raises
    ``ResolutionError`` if there is no script by that name in the ``scripts``
    metadata directory.  `namespace` should be a Python dictionary, usually
    a module dictionary if the script is being run as a module.


Exceptions
==========

``pkg_resources`` provides a simple exception hierarchy for problems that may
occur when processing requests to locate and activate packages::

    ResolutionError
        DistributionNotFound
        VersionConflict
        UnknownExtra

    ExtractionError

``ResolutionError``
    This class is used as a base class for the other three exceptions, so that
    you can catch all of them with a single "except" clause.  It is also raised
    directly for miscellaneous requirement-resolution problems like trying to
    run a script that doesn't exist in the distribution it was requested from.

``DistributionNotFound``
    A distribution needed to fulfill a requirement could not be found.

``VersionConflict``
    The requested version of a project conflicts with an already-activated
    version of the same project.

``UnknownExtra``
    One of the "extras" requested was not recognized by the distribution it
    was requested from.

``ExtractionError``
    A problem occurred extracting a resource to the Python Egg cache.  The
    following attributes are available on instances of this exception:

    manager
        The resource manager that raised this exception

    cache_path
        The base directory for resource extraction

    original_error
        The exception instance that caused extraction to fail


Supporting Custom Importers
===========================

By default, ``pkg_resources`` supports normal filesystem imports, and
``zipimport`` importers.  If you wish to use the ``pkg_resources`` features
with other (PEP 302-compatible) importers or module loaders, you may need to
register various handlers and support functions using these APIs:

``register_finder(importer_type, distribution_finder)``
    Register `distribution_finder` to find distributions in ``sys.path`` items.
    `importer_type` is the type or class of a PEP 302 "Importer" (``sys.path``
    item handler), and `distribution_finder` is a callable that, when passed a
    path item, the importer instance, and an `only` flag, yields
    ``Distribution`` instances found under that path item.  (The `only` flag,
    if true, means the finder should yield only ``Distribution`` objects whose
    ``location`` is equal to the path item provided.)

    See the source of the ``pkg_resources.find_on_path`` function for an
    example finder function.

``register_loader_type(loader_type, provider_factory)``
    Register `provider_factory` to make ``IResourceProvider`` objects for
    `loader_type`.  `loader_type` is the type or class of a PEP 302
    ``module.__loader__``, and `provider_factory` is a function that, when
    passed a module object, returns an `IResourceProvider`_ for that module,
    allowing it to be used with the `ResourceManager API`_.

``register_namespace_handler(importer_type, namespace_handler)``
    Register `namespace_handler` to declare namespace packages for the given
    `importer_type`.  `importer_type` is the type or class of a PEP 302
    "importer" (sys.path item handler), and `namespace_handler` is a callable
    with a signature like this::

        def namespace_handler(importer, path_entry, moduleName, module):
            # return a path_entry to use for child packages

    Namespace handlers are only called if the relevant importer object has
    already agreed that it can handle the relevant path item.  The handler
    should only return a subpath if the module ``__path__`` does not already
    contain an equivalent subpath.  Otherwise, it should return None.

    For an example namespace handler, see the source of the
    ``pkg_resources.file_ns_handler`` function, which is used for both zipfile
    importing and regular importing.


IResourceProvider
-----------------

``IResourceProvider`` is an abstract class that documents what methods are
required of objects returned by a `provider_factory` registered with
``register_loader_type()``.  ``IResourceProvider`` is a subclass of
``IMetadataProvider``, so objects that implement this interface must also
implement all of the `IMetadataProvider Methods`_ as well as the methods
shown here.  The `manager` argument to the methods below must be an object
that supports the full `ResourceManager API`_ documented above.

``get_resource_filename(manager, resource_name)``
    Return a true filesystem path for `resource_name`, coordinating the
    extraction with `manager`, if the resource must be unpacked to the
    filesystem.

``get_resource_stream(manager, resource_name)``
    Return a readable file-like object for `resource_name`.

``get_resource_string(manager, resource_name)``
    Return a string containing the contents of `resource_name`.

``has_resource(resource_name)``
    Does the package contain the named resource?

``resource_isdir(resource_name)``
    Is the named resource a directory?  Return a false value if the resource
    does not exist or is not a directory.

``resource_listdir(resource_name)``
    Return a list of the contents of the resource directory, ala
    ``os.listdir()``.  Requesting the contents of a non-existent directory may
    raise an exception.

Note, by the way, that your provider classes need not (and should not) subclass
``IResourceProvider`` or ``IMetadataProvider``!  These classes exist solely
for documentation purposes and do not provide any useful implementation code.
You may instead wish to subclass one of the `built-in resource providers`_.


Built-in Resource Providers
---------------------------

``pkg_resources`` includes several provider classes that are automatically used
where appropriate.  Their inheritance tree looks like this::

    NullProvider
        EggProvider
            DefaultProvider
                PathMetadata
            ZipProvider
                EggMetadata
        EmptyProvider
            FileMetadata


``NullProvider``
    This provider class is just an abstract base that provides for common
    provider behaviors (such as running scripts), given a definition for just
    a few abstract methods.

``EggProvider``
    This provider class adds in some egg-specific features that are common
    to zipped and unzipped eggs.

``DefaultProvider``
    This provider class is used for unpacked eggs and "plain old Python"
    filesystem modules.

``ZipProvider``
    This provider class is used for all zipped modules, whether they are eggs
    or not.

``EmptyProvider``
    This provider class always returns answers consistent with a provider that
    has no metadata or resources.  ``Distribution`` objects created without
    a ``metadata`` argument use an instance of this provider class instead.
    Since all ``EmptyProvider`` instances are equivalent, there is no need
    to have more than one instance.  ``pkg_resources`` therefore creates a
    global instance of this class under the name ``empty_provider``, and you
    may use it if you have need of an ``EmptyProvider`` instance.

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

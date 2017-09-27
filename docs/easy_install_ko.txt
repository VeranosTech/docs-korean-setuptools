============
Easy Install
============

Easy Install는 자동으로 파이썬 패키지를 다운로드, 빌드, 설치 그리고 관리해주는 ``setuptools`` 와 함께 묶인 파이썬 모듈이다(``easy_install``).

패키지를 설치하는 과정에서 어려움을 겪었다면 `distutils mailing list
<http://mail.python.org/pipermail/distutils-sig/>`_ 을 통해 연락을 해라. 
(setuptools의 저자들에게 직접적으로 이메일을 보내지는 말라. 그렇게 하면 이메일이 처리되지 않을 것이다. 
메일 리스트는 이전에 묻고 답한 질문에 대해서 검색이 가능한 아카이브이다. 
버그에 대해서 리포트 하기 전에 조사를 해보아야 한다. )

(또한, EasyInstall을 통해 당신의 패키지가 더 잘 작동하게 만들거나 사용자들에게 EasyInstall을 직접 사용하라는 요구 없이 EasyInstall 같은 기능을 제공하기 위해 ``setuptools`` 를 어떻게 사용할 수 있는지 배우고 싶다면, `setuptools`_ 문서를 확인하고 싶을 것이다.)


.. contents:: ** 목차 **


"Easy Install" 사용하기 
=========================


.. _installation instructions:

"Easy Install" 설치하기
-------------------------

지원되는 플랫폼에 대한 다운로드 링크와 기본 설치 지시 사항을 위해 `setuptools PyPI page <https://pypi.python.org/pypi/setuptools>`_ 를 봐라.

적어도 파이썬 2.6 버전이 필요하다. ``easy_intall`` 스크립트는 플랫폼 위의 파이썬 스크립트의 일반적인 위치에 설치될 것이다. 

setuptools PyPI 페이지에서 지시사항은 당신이 파이썬의 주요한 ``site-packages`` 디렉토리를 설치하고 있다는 것을 가정한다. 그렇지 않으면, 설치 전에, 아래에 있는 섹션인 `커스텀 설치 위치`_ 을 살펴봐야한다.
(윈도우에서 다른 위치에서 설치를 진행할 때 ``.exe`` 인스톨러를 사용하지 않아도 된다.)

``easy_install`` 은 일반적으로 인터넷에서 파일을 다운로드함으로써 작동한다.
파이썬 프로그램이 넷에 직접 접근하는 것을 막는 NTLM 기반의 방화벽이 있다면 `APS proxy server <http://ntlmaps.sf.net/>`_ 를 먼저 설치하고 사용해야할 수도 있다. 이를 사용해서 웹 브라우저와 동일한 방식으로 방화벽을 통과할 수 있게 해준다.

(대안으로, 실제로 다운로드를 할 때 easy_install 사용을 원치 않는다면, ``--allow-hosts`` 옵션으로 이를 제한할 수 있다. 더 자세한 사항을 위해서 `--allow-hosts 를 이용한 다운로드 제한`_ 과 `커맨드 라인 옵션`_ 섹션을 봐라.)


트러블 슈팅
~~~~~~~~~~~~~~~


EasyInstall/setuptools 정확하게 설치된다면, ``easy_install`` 명령을 수행할 수 있다. 그렇지 않다면 ``ImportError`` 에러가 발생할 것이다. 이 에러는 아래의 `커스텀 설치 위치`_ 에서 설명된 단계를 따르지 않고 ``site-packages`` 가 아닌 다른 곳에 설치할 경우 주로 발생한다. 
커스텀 위치가 제대로 작동하게 하기 위해서는 해당 섹션과 각 단계를 따라가라. 그리고 나서 재설치를 한다.  

유사하게, ``easy_install`` 을 실행하고 패키지 설치를 하는 것처럼 나타지만 임포트할 수 없다면, 이 경우는 EasyInstall은 정확하게 설치되었지만 준비가 되지 않은 표준 위치가 아닌 곳에 패키지를 설치한 경우가 많다. 
더 자세한 사항에 대해서는 아래의 `커스텀 설치 위치`_ 을 봐라.


윈도우 노트 
~~~~~~~~~~~~~

setuptools를 설치하는 것은  `실행 파일 및 실행 프로그램`_ 에 설명된 기술에 따라 ``easy_install`` 명령을 제공할 것이다.
``easy_install`` 명령은 설치 후에는 이용 불가능하다면, 해당 섹션은 윈도우에서 명령을 이용 가능하게 하기 위한 조작 방법에 대한 상세사항을 제공한다.


패키지 다운로드와 설
------------------------------------


``easy_install`` 의 기본 사용법을 위해, 소스 배포 또는 .egg file (`Python Egg`__) 의 파일 이름 또는 URL을 제공할 필요가 있다.

__ http://peak.telecommunity.com/DevCenter/PythonEggs


**Example 1**. 이름으로 패키지를 설치하고, PyPI에서 최신 버전을 검색하고 자동으로 다운로드, 빌드 및 설치를 한다.::

    easy_install SQLObject

**Example 2**. "download page" 에서 링크를 찾아 이름과 버전을 통해서 설치 또는 업그레이드를 한다.::

    easy_install -f http://pythonpaste.org/package_index.html SQLObject

**Example 3**. 지정된 URL에서 소스 배포판을 다운로드하여 자동으로 빌드하고 설치한다.::

    easy_install http://example.com/path/to/MyPackage-1.2.3.tgz

**Example 4**. 이미 다운로드된 .egg file을 설치한다.::

    easy_install /my_downloads/OtherPackage-3.2.1-py2.3.egg

**Example 5**. 이미 설치된 패키지를 PyPI에 나열된 최신 버전으로 업그레이드한다.::

    easy_install --upgrade PyProtocols

**Example 6**.  현재 디렉토리에서 이미 다운로드 되고 추출된 소스 배포를 설치한다.(New in 0.5a9)::

    easy_install .

**Example 7**.  (New in 0.6a1) 패키지에 대한 소스 배포와 하위버전 체크아웃 URL를 찾는다. 그리고 
그것을 추출하거나 ``~ / projects / sqlobject`` (이름은 항상 모두 소문자이다.)에 체크 아웃한다.
이를 통해 검사하거나 수정할 수 있다. (패키지는 설치되지 않을 것이지만 ``easy_install ~/projects/sqlobject``  를 통해 쉽게 설치된다. 더 자세한 사항을 위해서는 `소스 패키지 수정 및 보기`_ 를 봐라.)::

    easy_install --editable --build-directory ~/projects SQLObject

**Example 7**. (New in 0.6.11) 홈 디렉토리 내에서 배포를 설치한다.::

    easy_install --user SQLAlchemy


Easy Install은 URL, 파일이름, PyPI 패키지 이름( ``distutils`` "distribution" 이름)과 패키지+버전의 형태도 받아 들인다. 다른 경우에는 기준을 만족시키는 이용 가능한 최신 버전의 위치를 알아내려고 시도할 것이다.

다운로드하거나 다운로드된 파일을 처리할 때, Easy Install은 .tgz, .tar, .tar.gz,
.tar.bz2, 또는 .zip.의 확장자를 가지고 있는 distutils 소스 배포 파일을 인식한다. 그리고 이미 빌드된 .egg 배포와 distutils을 사용해서 빌드된 ``.win32.exe`` 을 처리한다. 

다른 디렉토리를 지정하기 위해 ``-d`` 또는 ``--install-dir`` 옵션을 제공하지 않거나 distutil configuration 파일(아래의 `Configuration Files`_, 참조)을 사용한 다른 위치를 지정하지 않으면 기본적으로 패키지는 실행되고 있는 파이썬 설치 ``site-packages`` 디렉토리에 설치된다. 

기본적으로 패키지에 포함된 스크립트는 실행중인 파이썬 설치 표준 스크립트 설치 위치에 설치된다. 그러나, 커맨드 라인이나 config 파일을 통해 위치를 지정한다면, 설치 스크립트의 기본 디렉토리는 스크립트가 설치된 패키지에 접근할 것을 보장하기 위해 패키지 설치 디렉토리와 같아질 것이다. ``-s`` 또는 ``--script-dir`` 옵션을 통해 오버라이드 할 수 있다. 

파이썬이 설치된 패키지는 설치 디렉토리 내의 ``easy_isntall.pth`` 파일에 추가된다. 
그래서 파이썬은 항상 가장 최근에 설치된 패키지 버전을 사용할 수 있다. 실행시 어떤 버전을 사용할지 선택하고 싶다면, 
``-m`` 또는  ``--multi-version`` 옵션을 사용해라.


패키지 업그레이드
-------------------

패키지 업그레이드를 위해 특별한 것을 할 필요는 없다. 단지 새로운 버전을 설치하거나 특정 버전을 요청하면 된다.
예를 들면 아래와 같다. ::

    easy_install "SomePackage==2.0"

지금 가지고 있는 버전보다 최신의 버전의 경우::

    easy_install "SomePackage>2.0"

PyPI에서 가장 최신 버전을 찾기 위해 업그레이드 flag를 사용한다 ::

    easy_install --upgrade SomePackage

또는 다운로드 페이지, 직접 다운로드 URL, 또는 패키지 파일 이름을 사용한다. ::

    easy_install -f http://example.com/downloads ExamplePackage

    easy_install http://example.com/downloads/ExamplePackage-2.0-py2.4.egg

    easy_install my_downloads/ExamplePackage-2.0.tgz

``-m`` 또는 ``--multi-version`` 을 사용한다면, 실행시 자동으로 ``require()`` 함수를 사용하는 것은 기준을 만족시키는 가장 최근에 설치된 패키지의 버전을 선택한다. 그래서, 새로운 버전을 설치하는 것은 단지 업그레이드를 하는 데 필요한 단계이다.

PYTHONPATH에 있는 디렉토리 또는 configured "site" 디렉토리에 설치를 하고 있다면, 패키지를 설치하는 것은 자동으로 ``easy-install.pth`` 내에 있는 이전 버전으로 대체될 것이다. 그래서 파이썬은 기본적으로 가장 최근에 설치된 버전을 임포트할 것이다. 다시 말하자면, 새로운 버전을 설치하는것은 단지 요구 되어지는 업그레이드 단계이다. 


스크립트 설치를 억제하지 않는 경우( ``--exclude-scripts`` 또는 ``-x`` 를 사용해서 ), 업그레이드된 버전의 스크립트는 설치될 것이다. 그리고 스크립트는 자동으로 상응하는 패키지 버전의 ``require()`` 에 패치될 것이다.  그래서 멀티버전모드로 설치되었더라도 사용에는 문제가 없다. 

``easy_install`` 는 패키지를 삭제하지 않는다. ( 존재하는 패키지와 같은 이름이나 같은 버전 넘버로 설치하지 않는 이상)
그래서 이전 버전의 패키지를 삭제하고 싶다면, 아래의  `패키지 삭제`_, 를 참고해라.
 


활성 버전을 변경하기 
---------------------------

패키즈를 업그레이드 하고 싶지만, 이전에 설치된 버전으로 돌아갈 필요가 있다면, 다음과 같이 할 수 있다.:: 

    easy_install PackageName==1.2.3

``1.2.3`` 은 변경하기를 원하는 정확한 버전 넘버로 대체된다. 요청된 이름과 버전과 매칭된 패키지가 ``sys.path`` 의 디렉토리에 설치되지 않았다면, PyPI를 통해 위치를 알아내고 설치될 것이다. 

가장 최근에 설치된 ``PackageName`` 버전으로 변경하려면 다음과 같이 하면 된다.:;

    easy_install PackageName

이는 최근에 설치된 버전을 활성화 할 것이다. ( distutils configuration 파일을 통해 ``find_links`` 를 설정한다면, 다운로드 페이지는 패키지의 가장 최근 버전을 체크할 것이고 현재 버전보다 최신의 버전이 있다면 그 버전을 다운로드, 설치를 할 것이다.)

``--exclude-scripts`` 또는 ``-x`` option 으로 지정되지 않았다면 활성화 패키지 버전을 번경하는 것은 새롭게 활성화된 버전의 스크립트를 설치할 것이다. 


패키지 삭제 
---------------------

패키지를 다른 버전으로 대체한다면, 당신은 단지 PackageName-versioninfo.egg 파일 또는 디렉토리 (설치 디렉토리에서 찾을 수 있음)를 삭제함으로써 필요하지 않는 패키지 버전을 삭제할 수 있다. 

현재 설치된 패키지 버전 또는 패키지의 모든 버전을 삭제하고 싶다면, 우선 이렇게 실행해야한다.::

    easy_install -m PackageName

이는 파이썬이 제거하기로 계획중인 패키지를 계속해서 검색하지 않는 것을 보장한다. 그 후에, 제거하기를 희망하는 스크립트를 따라 .egg 파일 또는 디렉토리를 안전하게 제거할 수 있다. 


스크립트 관리
----------------

``-x`` 또는 ``--exclude-scripts`` 옵션을 설정하지 않으면, 설치, 업그레이드, 패키지 버전 변경을 할 때마다, EasyInstall은 자동으로 선택된 패키지 버전의 스크립트를 설치한다. 스크립트 디렉토리에 있는 스크립트들이 같은 이름은 가진다면, 덮어 쓰여진다.

그러므로, 새로운 버전의 패키지가 같은 이름으로 된 스크립트를 포함하지 않지 않는 한, 수동으로 이전 패키지 버전의 스크립트를 수동으로 삭제할 필요가 없다. 그러나 완전히 패키지를 삭제하려면, 수동으로 스크립트를 삭제해야한다.

EasyInstall의 기본 동작은 한 번에 한 버전의 패키지에서만 스크립트를 정상적으로 실행할 수 있음을 의미한다.
그러나 여러 버전의 스크립트를 사용할 수 있게 하려면 ``--multi-version`` 또는 ``-m`` 옵션을 사용하고 EasyInstall이 생성하는 스크립트의 이름을 변경하면 됩니다.

EasyInstall 스크립트는가 나온 패키지의 일치하는 버전을 ``require()`` 해야하는 짧은 코드 stub으로 스크립트를 설치하기 때문에 이는 작동한다. 따라서 스크립트의 이름을 재명명하는 것은 아무런 영향이 없다.

예를 들면, `docutils <http://docutils.sf.net/>`_ 패키지에서 제공되는 ``rst2html`` 툴의 두 가지 버전을 사용하고 싶다고 가정하자. 먼저 한 가지 버전에 대해서 설치를 할 수 있다.::

    easy_install -m docutils==0.3.9

``rst2html.py`` 를 ``r2h_039`` 로 재명명한다. 그리고 다른 버전을 설치한다.::

    easy_install -m docutils==0.3.10

이는 다른 ``rst2html.py`` 스크립트를 생성할 것이다. 이 버전은 0.3.9 대신 0.3.10 버전의 docutils를 사용한다.
이제 두 가지 스크립트를 가지고 있고 각각은 다른 버전의 패키지를 사용한다. ( 두 가지 설치에 대해 ``-m`` 옵션을 사용하면, 파이썬은 가장 최근에 설치한 패키지 버전 이외의 것을 사용하지 못하게 한다.)



실행 파일 및 실행 프로그램
------------------------------

유닉스 시스템에서, 스크립트는 "#!" 헤더가 있고 확장자가 없고 헤더에 표시된 Python 버전으로 시작된다.

윈도우에서는 확장명이 없는 파일을 실행하는 매커니즘이 있다. 그래서 EasyInstall은 유닉스의 동작은 미러링 하는 두 가지 기술을 제공한다. 이 동작은 SETUPTOOLS_LAUNCHER 환경 변수에 의해 표시되고 이는 "executable" 또는 "natural" 일 수 있다.

사용된 기술에 관계 없이 스크립트는 스크립트 디렉토리(기본적으로 파이썬 설치 디렉토리)에 설치될 것이다. 
이 디렉토리가 PATH 환경 변수에 있는지 확인하는 것이 EasyInstall에 추천된다.
스크립트 디렉토리가 PATH에 있는지 확인하기 가장 쉬운 방법은 파이썬 디렉토리에서 ``Tools\Scripts\win_add2path.py`` 를 실행하는 것이다.
 
파이썬 스크립트 디렉토리를 포함하는 ``PATH`` 를 변경하는 것 대신에, 스크립트를 위한 설치 위치를 재설정할 수도 있다. 
그래서 이미 ``PATH`` 에 있는 디렉토리를 이동할 수 있다.
더 많은 정보를 알고 싶다면, `커맨드 라인 옵션`_ 와 `Configuration Files`_ 를 봐라.  
설치를 하는 동안, ``--script-dir`` 와 같은 커맨드 라인 옵션은 ``ez_setup.py`` 에 전달된다. 그리고 ``easy_install.exe`` 가 설치될 곳을 관리한다.


윈도우 실행 파일 프로그램 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

실행 파일 프로그램이 사용된다면, EasyInstall은 설치된 각 스크립트 옆에 있는 같은 이름의 스크립트'.exe' 런처를 만들 것이다.( ``easy_install`` 도 포함). 이러한 작은 .exe 파일은 '#!' 헤더에서 표시되는 파이썬 버전을 사용하여 같은 이름의 스크립트를 시작할 것이다.
이 동작은 현재 기본 설정값이다. 실행 파일 프로그램을을 강제로 사용하려면 ``SETUPTOOLS_LAUNCHER`` 을 "executable"로 설정해라.


Natural Script Launcher
~~~~~~~~~~~~~~~~~~~~~~~

EasyInstall은 또한 스크립트를 시작하기 위한 `pylauncher <https://bitbucket.org/pypa/pylauncher>`_ 와 같은 외부 런처로 실행 지연을 지원한다. 
``SETUPTOOLS_LAUNCHER`` 환경 변수를 "natural"로 설정하여 실험적인 기능을 가능하게 해라. 그리고 나서 EasyInstall은 .pya(또는 .pyw) 확장자가 추가된 간단한 스크립트로 스크립트를 설치한다.
이러한 확장이 pylauncher와 연관이 있고 PATHEXT 환경 변수에 나열되어 있으면, 이러한 스크립트는 다른 실행 파일들 처럼 간단하고 직접적으로 호출될 수 있다. 이 동작은 향후 버전에서 기본값이 될 수도 있다.

EasyInstall은 전형적인 '.py' 확장자 대신에 .pya 확장을 사용한다. 이렇게 구별되는 확장자는 파이썬이 스크립트를 임포트 가능한 모듈(이름 충돌이 존재하는 곳)로 취급하지 못하도록 하는데에 필요하다.
현재 pylauncher의 릴리즈는 아직 .pya 파일을 기본값으로 설정하지 않는다. 그러나 향후에는 그렇게 될 것이다.


팁과 기술
-----------------

다중 파이썬 버전
~~~~~~~~~~~~~~~~~~~~~~~~

EasyInstall은 EasyInstall 자체를 두 가지 이름으로 설치한다.:

``easy_install`` 과 ``easy_install-N.N`` . ``N.N`` 은 이를 설치하기 위해 사용되는 파이썬 버전이다. 
그러므로 파이썬3.2와 2.7에 모두에 대해 EasyInstall을 설치한다면, 개별 파이썬 버전에 대해 패키지를 설치하기 위해  `` easy_install-3.2`` 또는 ``easy_install-2.7`` 스크립트를 사용할 수 있다.

Setuptools는 또한 설치된 Setuptools를 가지고 있는 파이썬에 대해 ``python -m easy_install`` 을 사용해서 발생하는 실행가능한 모듈로서 easy_install을 제공한다. 


``--allow-hosts`` 를 이용한 다운로드 제한
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``--allow-hosts`` (``-H``) 옵션을 사용하면 EasyInstall이 링크와 다운로드는 찾을 도메일을 제한할 수 있다.
``--allow-hosts=None`` 은 다운로드를 완전히 막는다. 
또한 와일드 카드를 사용하여 자신의 인트라넷에 있는 호스트에 다운하는 것을 제한할 수 있다.
``--allow-hosts``  옵션에 대한 자세한 정보를 위해서는 `커맨드 라인 옵션`_ 를 봐라.

기본적으로 호스트 제한이 없지만, 적절한 `configuration files`_ 을 수정하는 것과 추가하는 것을 통해 이 기본값을 변경할 수 있다.:

.. code-block:: ini

    [easy_install]
    allow_hosts = *.myintranet.example.com,*.python.org

위의 예는 커맨드 라인에서 덮어 쓰지 않는 한, ``python.org`` 와 ``myintranet.example.com`` 도메인에 있는 호스트들로부터 다운로드만을 허락한다.


Un-networked Machines에서의 설치 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

필요한 에그나 소스 패키지를 해당 머신의 디렉토리에 복사하기만 하면 된다. 
그리고 나서 디렉토리의 위치를 지정하기 위해 ``-f`` 또는 ``--find-links`` 옵션을 사용한다. 예를 들면 ::
 
    easy_install -H None -f somedir SomePackage

위는 단지 ``somedir`` 에서 찾은 에그와 소스 패키지를 사용하게 SomePackage르 설치를 시도하고 모든 원격 접근을 허락하지 않는다. somdir에서 SomePackage의 종속성을 모두 사용할 수 있어야 한다.

같은 운영체제와 라이브러리 버전의 다른 머신이 있다면(또는 패키지의 플랫폼이 지정되지 않았다면) 다음과 같은 명령을 통해 egg의 디렉토리를 만들 수 있다.::

    easy_install -zmaxd somedir SomePackage

이는 EasyInstall이 스크립트나 .pth 파일을 만들지 않고 SomePakage에 대한 압축된 에그 또는 소스 패키지와 모든 종속성을 ``somedir`` 에 넣도록 한다. ``somedir`` 의 내용을 대상 머신에 복사할 수 있다.
( ``-z`` 는 압축된 에그를 뜻하고 ``-m`` 은 .pth 파일을 사용하지 못하게 하는 다중 버전을 의미한다. ``-a`` 는 설치되어 있음에 불구하고, 필요한 모든 에그의 복사를 의미한다. ``-d`` 는 에그를 놓을 디렉토리의 위치를 표시한다.)

또한 ``-l`` 옵션을 포함해서 `setup.py develop` 명령으로 설치된 로컬 개발 패키지로부터 에그를 빌드할 수 있다. 
예를 들면 ::

    easy_install -zmaxld somedir SomePackage

이것은 로컬로 이용 가능한 소스 배포판을 사용하여 에그를 빌드한다.  


에그로 다른 프로젝트를 패키징하기 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

에그의 형태로 게시되지 않은 패키지를 배포할 필요가 있는가? 프로젝트에 대한 에그를 빌드하기 위해 EasyInstall을 사용할 수 있다.
``--zip-ok``, ``--exclude-scripts``, ``--no-deps`` 옵션을 (``-z``, ``-x`` and ``-N``, respectively) 를 사용하기를 원할 것이다. 에그를 넣을 위치를 지정하기 위해 ``-d`` 또는 ``-install-dir`` 을 사용해라. 웹에 게시된 디렉토리에 에그를 넣음으로써, 인트라넷 또는 인터넷에서 에그를 다운로드 할 수 있다.

누군가 단일 ``.py`` 파일의 형태로 패키지를 배포한다면, 파일의 URL에  ``#egg=name-version`` 접미사를 붙임으로써 에그 안에서 래핑할 수 있다.
다음 예시와 같다. ::

    easy_install -f "http://some.example.com/downloads/foo.py#egg=foo-1.0" foo

에그로 패키지를 인스톨 할 것이다. ::

    easy_install -zmaxd. \
        -f "http://some.example.com/downloads/foo.py#egg=foo-1.0" foo

이는 ``.egg`` 파일을 현재 디렉토리에 만들 것이다.


패키지 인덱스 만들기 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

로컬 디렉토리와 파이썬 패키지 인덱스 외에도, EasyInstall은 ``-f`` ( ``-find-link`` )옵션을 사용해서 웹페이지에서 다운로드 링크를 발견할 수 있다. 아주 간단한 예시로, 에그나 파이썬 소스 패키지에 대한 링크가 있는 웹 페이지, 심지어 자동으로 생선된 디렉토리 목록(Apache 웹 서버가 제공하는)을 가질 수 있다. 

패키지 다운로드를 위한 인트라넷 사이트를 설정하고 있다면, 기본적으로 다운로드 사이트를 사용하도록 대상 머신을 구성하고 이 파일을 `Configuration files`_ 에 추가할 수 있다. :

.. code-block:: ini

    [easy_install]
    find_links = http://mypackages.example.com/somedir/
                 http://turbogears.org/download/
                 http://peak.telecommunity.com/dist/


보다시피, 공백으로 구분된 여러 개의 URL을 나열할 수 있다. 필요한 경우 여러 줄로 계속 나열 할 수 있다. ( 후속 줄을 들여쓰기 하는 한)

더 야망이 있다면 전체 커스텀 패키지 인덱스 또는 PyPI 미러를 만들 수도 있다. 
아래의 `커맨드 라인 옵션`_ 내의 ``--index-url`` 옵션과 `패키지 인덱스 "API"`_ 섹션을 봐라.


패스워드로 보호된 사이트
------------------------

니가 다운로드 하기를 원하는 사이트가 HTTP "Basic" 권한을 통해 패스워드로 보호되어 있다면, 아래와 같은 방법으로 URL에서 인증서를 지정할 수 있다. ::

    http://some_userid:some_password@some.example.com/some_path/

인덱스 페이지 URL과 직접 다운로드 URL로 이를 할 수 있다. easy_install로 읽은 HTML 페이지가 다운로드 지점에 상대 링크를 사용하는 한, 같은 사용자 ID 또는 패스워드는 다운로드를 하는데 사용될 것이다.

.pypirc 인증서 사용하기
-------------------------

URL에 보증서를 제공하는 것 외에도 ``easy_install`` 또한 .pypirc 파일에 인증서가 있는 경우 인증서를 사용한다.
패키지의 개인 저장소를 관리하는 팀은 이미 distutils 문서에 따라 패키지를 업로드 하기 위한 액세스 인증서를 정의했을 수도 있다. 
``easy_install`` 은 존재하는 경우, 이를 존중한다. sysntax에 대한 자세한 내용은 Python 2.5 이상의 distutils 문서를 참고해라. 


빌드 옵션 제어
~~~~~~~~~~~~~~~~~~~~~~~~~

EasyInstall은 표준 distutils `Configuration Files`_ 을 존중한다. 그래서 소스로부터 설치된 패키지를 위한 빌드 옵션을 구성하게 위해 이러한 파일을 사용할 수 있다. 예를 들면, 윈도우에서 MinGW 컴파일러를 사용한다면, 이렇게 함으로써 기본 컴파일러를 구성할 수 있다.:

.. code-block:: ini

    [build]
    compiler = mingw32

적절한 distutils 설정 파일에 넣는다. 사실, 이것은 정상적인 distutils 구성일 뿐이기 때문에, EasyInstall 에 의해 수행되는 것 뿐만 아니라 해당 config 파일을 사용하는 모든 빌드에 영향을 미친다. 예를 들면, ``distutils`` 패키지 디렉토리에 있는 ``distutils.cfg`` 에 이러한 줄을 더한다면, 빌드하는 모든 패키지에 대한 컴파일러가 될 것이다. 

표준 설정 파일 위치 목록과 distutils 설정 파일 사용에 대한 더 많은 문서 링크가 알고 싶다면 `Configuration Files`_ 을 봐라.


소스 패키지 수정 및 보기
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

때때로 패키지의 소스 배포판에는 실제 코드의 일부가 아닌 추가 문서, 예제, 구성 파일 등이 포함되어 있다.
이 파일들은 검사할 수 있도록 하기 위해, ``--editable`` 옵션을 EasyInstall에 사용할 수 있다. 
그리고 EasyInstall은 소스 배포판 또는 패키지의 하위버전 URL을 찾을 것이다. 그런 다음, 다운로드를 하고 추출을 하거나 지정한 ``--build-directory`` 의 하위 디렉토리로 체크를 할 것이다.  
그리고 다음 패키지를 수정 또는 구성한 후, 설치하려면 해당 디렉토리에 EasyInstall을 재실행하여서 패키지를 설치할 수 있다.

``--editable`` 을 사용하려면 EasyInstall이 실제로 패키지를 빌드하거나 설치하는 것을 멈춘다. 이는 단지 찾고, 얻고, 언팩을 가능하게 해준다. 

필요한 경우 패키지를 변경하거나 ``setup.py develop`` (패키지가 setuptools를 사용한다면)을 사용해서 개발 환경을 설치하는 것을 허락한다. 또는 ``easy_install projectdir`` (``projectdir`` 은 다운로드 패키지를 위해 만들어진 EasyInstall 하위 디렉토리)을 실행시키는 것도 허락한다.

``--editable`` (줄여서 ``-e``) 를 사용하기 위해, ``--build-directory`` (줄여서 ``-b``) 를 사용해야한다. 
프로젝트는 빌드 디렉토리의 서브디렉토리 내에 위치할 것이다. 서브디렉토리는 프로젝트와 같은 이름을 가지지만 모두 소문자로 표시될 것이다. 파일 또는 디렉토리의 이름이 이미 조냊한다면, EasyInstall은 에러 메시지를 발생시키고 종료 시킬 것이다.

또한 ``--editable`` 을 사용할 때, URL 또는 파일이름을 argument로 사용할 수 없다. 
EasyInstall이 어떤 디렉토리가 만들어졌는지 확인하기 위해 프로젝트의 이름(그리고 선택적인 버전 요구사항)을 지정해야한다.
EasyInstall이 특정 URL이나 파일이름은 강제로 사용해야한다면, ``--find-links`` (줄여서 ``-f``)아이템으로 지정해야한다. 그리고 또한 프로젝트의 이름도 지정해야한다. 예를 들면 ::
 
    easy_install -eb ~/projects \
     -fhttp://prdownloads.sourceforge.net/ctypes/ctypes-0.9.6.tar.gz?download \
     ctypes==0.9.6


설치 충돌 문제 해결
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

(0.6a11 버전부터 이 섹션이 더 이상 사용되지 않는다. 이전 버전의 EasyInstall을 사용하는 사람들이 이를 참조 할 수 있도록 여기에 보관된다. 버전 0.6a11부터 설치 충돌은 이전 또는 시스템에 설치된 패키지를 삭제하지 않고 또한 문제도 무시하지 않고 자동으로 처리된다. 대신 egg는 ``easy-install.pth`` 파일에 추가 된 특별한 코드를 사용하여 자동으로 ``sys.path`` 의 앞 쪽으로 이동한다. 따라서 setuptools 버전 0.6a11 이상을 사용하는 경우 충돌에 대해 걱정할 필요가 없으며 다음 문제가 적용되지 않는다.)

EasyInstall은 "managed" 방법으로 배포판을 설치한다. 개별 배포판은 ``sys.path`` 에서 독립적으로 활성화 또는 비활성화 될 수 있다. 그러나 EasyInstall에 의해 설치되지 않은 패키지는 일반적으로 하나의 디렉토리에 모두 있으면 독립적으로 활성화되거나 비활성화 될 수 없다는 점에서 "unmanaged" 입니다.
 
따라서, EasyInstall을 사용하여 기존 패키지를 업그레이드하거나 기존 패키지와 동일한 이름의 패키지를 설치하는 경우, EasyInstall은 충돌을 경고를 할 것이다. 
(이는 ``setup.py install`` 보다 향상된 것이다. 왜냐하면 ``distutils`` 는 오래된 패키지 위에 새로운 패키지를 설치하기 때문이다. 관련이 없는 두 개의 패키지를 결합하거나 최신 버전의 패키지에서 삭제된 모듈을 남길 수 있다.)

EasyInstall은 기존의 "unmanaged" 패키지와 설치 중인 모든 배포판의 모듈 또는 패키지 간에 충돌을 발견하면 설치를 중지한다. 새로운 패키지가 제대로 작동하려면 삭제해야할 기존 파일과 디렉토리의 목록이 표시된다. 계속하려면 충돌하는 파일과 디렉토리를 수동으로 삭제하고 EasyInstall을 재실행해야한다. 

물론, 기존의 존재하는 "unmanaged" 패키지를 EasyInstall에서 "managed" 되는 버전으로 바꾸면 충돌에 대해서 더 이상 걱정할 필요가 없다.


압축 설치
~~~~~~~~~~~~~~~~~~~~~~~

EasyInstall은 가능한 경우, 압축 형식으로 패키지를 설치하려고 한다. 
zip 패키지는 ``--multi-version`` 옵션을 사용하지 않으면 파이썬의 전반적인 임포트 성능을 향상시킬 수 있다.
파이썬은 디렉토리보다 ``sys.path`` 에 있는 zip 파일 항목을 훨씬 빠르게 처리하기 때문이다.

0.5a9 버전부터 EasyInstall은 패키지를 분석하여 zip 파일로 안전하게 설치할 수 있는지 여부를 결정한 다음 분석을 수행한다. (이전 버전에서는 ``--zip-ok`` 옵션을 사용하지 않는 한, 패키지를 zip 파일로 설치하지 않는다.)

현재의 분석 접근 방법은 꽤 보수적이다. :

 * Any use of the ``__file__`` or ``__path__`` variables (which should be
   replaced with ``pkg_resources`` API calls)

 * Possible use of ``inspect`` functions that expect to manipulate source files
   (e.g. ``inspect.getsource()``)

 * Top-level modules that might be scripts used with ``python -m`` (Python 2.4)

설치 중인 패키지에서 위의 항목 중 하나라도 발견되면 EasyInstall은 패키지를 zip 파일에서 안전하게 실행할 수 없다고 가정하고 디렉토리에 압축을 푼다.
이 분석을 ``-zip-ok`` 플래그로 오버라이드 할 수 있다. 그러면 EasyInstall이 패키지를 zip 파일로 설치하도록 알려준다. 또는 ``--always-unzip`` 플래그를 사용할 수도 있다. 이는 EasyInstall이 분석 결과 패키지가 zip 파일로 실행하는 것이 안전하다고 하더라도 항상 압축을 푼다.


그러나 일반적으로 EasyInstall이 zip 또는 unzip 여부를 결정하고 문제를 해결하는 데 필요할 때만 재정의를 지정하는 것이 가장 간단하다. EasyInstall의 추측을 무시해야하는 경우, 패키지 저자와 EasyInstall 관리자에게 문의하여 향후 버전에서 적절하게 변경할 수 있다.

(패키지가 설정 스크립트에서 ``setuptools`` 를 사용한다면, 패키지 저자는 ``setup ()`` 에 대한 ``zip_safe``  argument를 통해 패키지를 안전하거나 안전하지 않은 것으로 선언할 수 있는 옵션을 갖게 된다. 패키지 저자가 이러한 선언을 하면 EasyInstall은 패키지 저자를 믿고 스스로 분석을 수행하지 않는다. 그러나 명령 줄 옵션이있는 경우 패키지 작성자의 선택보다 우선시 한다.)


참조 메뉴얼
================

Configuration Files
-------------------

(New in 0.4a2)

EasyInstall의 기본 옵션은 ``easy_install`` 이라는 제목 아래에 있는 표준 distutils 구성 파일을 사용하여 지정할 수 있다. EasyInstall은 먼저 현재 디렉토리에서 ``setup.cfg`` 파일을 찾은 다음  ``~/.pydistutils.cfg`` 또는 ``$HOME\\pydistutils.cfg`` (on Unix-like OSes and Windows, respectively) 그리고 마지막으로 ``distutils`` 패키지 디렉토리 내의 ``distutils.cfg`` 파일을 찾을 것이다. 여기에 간단한 예가 있다.:


.. code-block:: ini

    [easy_install]

    # set the default location to install packages
    install_dir = /home/me/lib/python

    # Notice that indentation can be used to continue an option
    # value; this is especially useful for the "--find-links"
    # option, which tells easy_install to use download links on
    # these pages before consulting PyPI:
    #
    find_links = http://sqlobject.org/
                 http://peak.telecommunity.com/dist/

EasyInstall은 ``[easy_install]`` 에서 자체 옵션에 대한 설정을 허용하는 것 외에도 다른 distutils 명령에 지정된 기본값을 존중한다.

예를 들어, ``[easy_install]`` 에 ``install_dir`` 을 설정하지 않고, ``[install]`` 명령에 대해 ``install_lib`` 를 설정했다면, 이는 EasyInstall의 기본 설치 디렉토리가 될 것이다.

그러므로, 이미 distutils 구성 파일을 사용해서 기본 설치 위치, 빌드 옵션 등을 설정했다면, EasyInstall은 ``[easy_install]`` 섹션에서 명시적으로 오버라이드 하지 않을 때까지 기본 설정을 존중한다.

더 많은 정보를 얻고 싶다면 `use and location of distutils configuration files <https://docs.python.org/install/index.html#inst-config-files>`_ 에서 현재 파이썬 문서를 봐라.

``easy_install`` 은 ``setup.py`` 에서 ``install_requires`` 옵션을 통해 촉진된 경우에만 현재 작업 디렉토리의 ``setup.cfg`` 를 사용할 것이다. 독립 실행 명령은 해당 파일을 사용하지 않는다.


커맨드 라인 옵션
--------------------

``--zip-ok, -z``
    zip 파일로 실행하기에 안전하지 않다고 표시되더라도 zip 파일로 모든 패키지를 설치해라. 이는 EasyInstall non-setuptools 패키지 분석이 너무 보수적이 일 때 유용할 수 있다. 하지만 패키지가 제대로 동작하지 않음을 명심해라. (0.5a9에서 변경, 이전에는 이 옵션이 압축 설치가 전혀 발생하지 않게 하기 위해 필요했음)

``--always-unzip, -Z``
    패키지가 zip 파일로 실행하기에 안전한 것으로 표시되더라도 zip 파일로 설치하지 마라. 이는 패키지가 안전하지 않은 일을 하지만 EasyInstall이 쉽게 감지할 수 없을 경우 유용하다. EasyInstall의 기본 분석은 매우 보수적이다. 그러나 특정 패키지에 문제가 있는 경우에만 이 옵션을 사용하고 패키지 관리자와 EasyInstall 관리자에게 문제를 보고 한 후에 이 옵션을 사용해야한다. 
    
    (노트 :``-z / -Z`` 옵션은 대상 디렉토리에 아직 설치되지 않은 체 새로 빌드되거나 다운로드 된 패키지의 설치에만 영향을 준다. 기존에 설치된 버전을 ZIP에서 UNZIP 파일로 변환하려는 경우 또는 그 반대의 경우, 먼저 기존 버전을 삭제하고 EasyInstall을 재실행 해야한다.)


``--multi-version, -m``
    다중 버전 모드. 이 옵션을 지정하면 ``easy_install`` 이 설치 중인 패키지에 대한 ``easy_intall.pth`` 항목을 추가하지 못한다. 그리고 패키지가 이미 존재하는 버전의 항목이 있으면, 이는 설치가 완료되면 제거된다.
    다중 버전 모드에서 ``pkg_resources.require()`` 를 사용하여 ``sys.path`` 에 넣지 않는 한, 특정 버전의 패키지를 임포트할 수 없다. 이것은 다음과 같이 간단하게 할 수 있다.::

        from pkg_resources import require
        require("SomePackage", "OtherPackage", "MyPackage")

    이는 ``sys.path`` 에 가장 최근에 설치된 특정 패키지의 버전을 놓을 것이다.
    (특정 버전을 선택하고 선택적인 종속성을 가능하게 하는 것과 같은 고급 사용에 대한 정보는 ``pkg_resources`` API 문서를 봐라.)
    
    0.6a10에서 변경됨 : non-PYTHONPATH 와 non-"site" 디렉토리에 설치될 때, 이 옵션은 더 이상 자동으로 가능하지 않다. 활성화를 원한다면 명시적으로 이 옵션을 사용해야한다.

``--upgrade, -U``   (New in 0.5a4)
    기본적으로 프로젝트/버전 요구사항이 sys.path 또는 설치 디렉토리에 이미 설치된 배포판에 의해 충족 시키지 못한다면, EasyInstall은 온라인을 검색한다. 그러나 ``--upgrade`` 또는 ``-U`` 플래그를 제공한다면, 설치를 위한 버전을 선택하기 전에 EasyInstall은 항상 패키지 인덱스와 ``--find-links`` URL 를 검사할 것이다. 
    이런 방법으로, EasyInstall이 가장 최근의 사용가능한 버전의 패키지를 사용할 수 있게 한다. 
    (이후 버전을 제외시킬 수있는 버전 요구 사항이있을 수 있음.)

``--install-dir=DIR, -d DIR``
    
    설치 디렉토리를 설정한다. 실행 시에 ``sys.apth`` 에 이 디렉토리가 있어야 한다. 그리고 필요한 설치된 패키지를 위해 ``pkg_resources.require()`` 를 사용해야한다.
    
    (0.4a2의 새로운 기능) 
    이 옵션을 커맨드라인이나 distutils 구성 파일에 직접 지정하지 않으면 distutils 기본 설치 위치가 사용된다. 일반적으로 이는 ``site-packages`` 디렉토리이겠지만, ``prefix`` 또는 ``install_lib`` 와 같은 것을 설정하는 distutils 설정 파일을 사용한다면, 기본 설치 디렉토리를 계산할 때 그 설정을 고려한다. 설치 디렉토리와 ``--prefix`` 라는 옵션이 있다. 

``--script-dir=DIR, -s DIR``

    스크립트 설치 디렉토리를 설정하십시오. 커맨드 라인이나 구성 파일을 통해 이 옵션을 사용하지 않지만, ``--install-dir`` 을 사용했다면, 이 옵션은 기본적으로 동일한 디렉토리에 있다. 그래서 스크립트가 관련 패키지  설치를 찾을 수 있을 것이다. 그렇지 않으면 distutils이 일반적으로 스크립트를 설치하는 위치에 대한 기본 설정은 distutils 구성 파일 설정을 고려할 것이다.
    
``--exclude-scripts, -x``
    스크립트를 설치하지 마라. 패키지의 여러 버전을 설치해야 하지만 이미 설치된 스크립트에 의해 실행될 버전을 재설정하고 싶지 않다면 이는 유용하다. 

``--user`` (New in 0.6.11)
    전역 site-packages 대신 : pep :`370` 에 지정된대로 user-site-packages를 사용해라.

``--always-copy, -a``   
    (0.5a4의 새로운 기능)
    필요한 모든 배포판을 sys.path의 디렉토리에 이미 있는 경우라도 설치 디렉토리에 복사해라.
    이전 버전의 EasyInstall에서는 이는 기본 동작이었지만 이제 명시적으로 요청해야 합니다.
    기본적으로 EasyInstall은 커맨드라인에서 배포판의 파일 이름을 명시적으로 지정하지 않으면 다른 배포를 sys.path 디렉토리의 배포 디렉토리에서 설치 디렉토리에 복사하지 않는다. 

    0.6a10에서 이 옵션을 사용하면 시스템과 개발 에그를 신뢰를 가지고 복사할 수 없으므로 고려 대상에서 제외된다. 
    이는 EasyInstall이 기대한 것보다 이전 버전을 선택하는 원인이 되거나 이미 설치된 것의 복사본을 다운로드하거나 설치하는 원인이 될 수도 있다. EasyInstall이 이전 버전으로 되돌아가거나 새로운 복사본을 다운로드 하기 전에, 건너 뛸 에그에 대한 경고 메세지가 표시된다.

    
``--find-links=URLS_OR_FILENAMES, -f URLS_OR_FILENAMES``
    특정 "다운로드 페이지"또는 디렉토리를 스캔하여 에그 또는 기타 배포본에 대한 직접 링크를 찾는다.
    기존 파일이나 디렉토리 이름 또는 직접 다운로드 URL은 EasyInstall의 검색 캐시에 즉시 추가되고 간접 URL(에그 또는 다른 인식된 보관 형식을 가리키지 않은 URL)은 추가 링크 목록에 추가되어 다운로드 링크를 검색한다. 
    EasyInstall이 패키지를 찾기 위해 온라인 상태가 되자마자(로컬에 존재하지 않거나 ``--upgrade`` 또는 ``U`` 가 옵션이 사용되었기 때문에), 지정된 URL은 추가적인 직접 링크에 대해 다운로드되고 스캔된다.

    ``-find-links`` 의 방법으로 발견된 에그와 아카이브는 커맨드 라인에서 지정된 요구사항을 만족시키기 위해 필요하다면 다운로드 되어진다. 불필요한 패키지에 대한 링크는 무시된다.
    
    모든 요청된 패키지가 다운로드 페이지에서 지정된 링크를 사용해서 발견된다면, ``--upgrade`` 또는 ``-U`` 옵션을 지정하지 않는 한, 파이썬 패키지 인덱스는 참조될 수 없다.

    (노트 : 링크를 포함하는 로컬 HTML을 참조하고 싶다면, 디렉토리, 에그를 참조하지 않거나 아카이브가 무시되는 파일 이름으로 ``file:`` URL 을 사용해야한다.)

    공백으로 구분된 이 옵션을 사용하여 여러 URL 또는 파일 또는 디렉토리의 이름을 지정할 수 있다.
    커맨드 라인에서, URL 목록을 따옴표로 묶어야 하므로 단일 옵션 값으로 인식된다. 또한 구성 파일내의 URL도 지정할 수 있다. 위의 `Configuration Files`_ 파일을 보자.
    
    0.6a10에서 변경사항 : 이전에 이 옵션을 사용한 모든 URL과 디렉토리는 가능한 빨리 스캔되었지만, 0.6a10 버전부터는 디렉토리와 직접 아카이브 링크만 즉시 스캔된다. 패키지 검색이 로컬에서 사용될 수 없거나 ``--update`` 또는 ``-U`` 옵션을 사용했기 때문에 이미 패키지 검색이 온라인 상태로 되지 않는 한 URL은 검색되지 않는다. 
    
``--no-find-links`` 
    링크 추가를 차단한다.
    easy_install이 설치되고 있는 프로젝트에서 정의된 링크가 추가되는 것을 막고 싶다면(요청된 프로젝트 또는 종속성인지 간에) 이 파라미터는 유용하다. 사용될 때, ``--find-links`` 는 무시된다.
    Distribute 0.6.11 과 Setuptools 0.7에서 추가되었다.
    
``--index-url=URL, -i URL`` (New in 0.4a1; default changed in 0.6c7)
    파이썬 패키지 인덱스의 기본 URL을 지정한다. 지정되지 않으면 기본값은 https://pypi.python.org/simple 이다.
    패키지가 ``--find-links`` 다운로드 페이지로부터 링크되거나 로컬에서 사용할 수 없는 패키지가 요청되었을 때,
    패키지 인덱스는 필요한 패키지에 대한 다운로드 페이지를 위해 검색될 것이다. 그리고 이러한 다운로드 페이지는 링크가 에그 또는 소스 배포판을 다운로드하기 위해 검색될 것이다.

``--editable, -e`` (New in 0.6a1)
    단지 지정된 프로젝트를 위해 소스 배포판을 찾고 다운로드한다. 그리고 지정된``--build-directory`` 의 서브 디렉토리에 압축을 해제한다.
    EasyInstall은 요청한 프로젝트나 그 의존성을 실제로 빌드하거나 설치하지는 않을 것이다. 단지 찾아내고 추출할 것이다. 자세한 사항이 궁금하다면, 위의 `소스 패키지 수정 및 보기`_ 를 봐라.

``--build-directory=DIR, -b DIR`` (UPDATED in 0.6a1)
    소스 패키지를 빌드하기 위해 사용된 디렉토리를 설정하기. 소스 배포나 체크아웃으로부터 패키지가 빌드 된다면, 지정된 디렉토리의 서브 디렉토리에 추출된다. 서브 디렉토리는 추출된 배포판의 프로젝트와 이름이 같지만 모두 소문자로 표시된다.
    해당 이름의 파일이나 디렉토리가 주어진 디렉토리에 이미 존재한다면, 콘솔에 경고가 표시될 것이다. 대신에 빌드는 임시 디렉토리에서 수행될 것이다.
    
    이 옵션은 EasyInstall이 단지 소스 배포판을 발견하고 추출하는 것(빌드나 설치는 아님)을 강요하는 ``--editable`` 옵션과 함께 가장 유용하게 사용된다.
    더 자세한 정보를 위해서는 위의 `소스 패키지 수정 및 보기`_ 를 봐라. 

``--verbose, -v, --quiet, -q`` (New in 0.4a4)
    EasyInstall의 진행 메시지 세부 정보 수준을 제어한다. 기본 상세 수준은 "info"이며, 이는 설정 스크립트 실행, 아카이브 압축 해체 또는 URL 검색과 같이 상대적으로 시간이 많이 소요되는 작업에 대해서만 정보를 출력한다.     
    ``-q`` 또는 ``--quiet`` 를 사용하는 것은 상세 수준이 "warn"으로 떨어지고 설치 보고서, 경고, 에러에 대해서만 표시할 것이다. ``-v`` 또는 ``--verbose`` 를 사용하면 상세 레벨이 올라가서 실행되는 모든 설정 스크립트의 개별 파일 수준 작업, 링크 분석 메시지 및 distutils 메시지가 포함된다. ``-v`` 옵션을 두 번 이상 포함 시키면 두 번째 및 뒤따라오는 사용은 모든 스크립트로 전달되어 보고의 상세도가 올라간다.

``--dry-run, -n`` (New in 0.4a4)
    실제로 패키지 또는 스크립트를 설치하지마라. 이 옵션은 실행되는 모든 설치 스크립트로 전달되므로 패키지가 실제로 빌드되지 않아야 한다. 이는 다운로드는 건너뛰지 않으며, 임시/빌드 디렉토리로 소스 배포판을 추출하는 것도 건너 뛰지 않는다. 

``--optimize=LEVEL``, ``-O LEVEL`` (New in 0.4a4)
    소스 배포판을 설치하고 있고, ``--zip-ok`` 옵션을 사용하고 있지 않다면, 이 옵션은 ``.py`` 파일에서 ``.pyo`` 파일로 컴파일링에 대한 최적의 수준을 제어한다.
    ``.egg`` 파일에 포함되어 있는 모듈의 컴파일에는 영향을 미치지 않고, ``.egg`` 디렉토리에 있는 모듈의 컴파일에만 영향을 미친다. 최적화 수준은 0,1 또는 2로 설정될 수 있다. 기본값은 0이다. 
    (distutils 구성 파일 중 하나에 ``install`` 또는 ``install_lib`` 로 설정되어 있지 않으면)
    

``--record=FILENAME``  (New in 0.5a4)
    
    설치된 모든 파일의 기록을 FILENAME에 기록해라. 이는 기본적으로 표준 distutils "install" 명령과 동일한 옵션이고 "setup.py install"에 이 옵션을 전달할 도구와의 호환성을 위해 포함된다. 

``--site-dirs=DIRLIST, -S DIRLIST``   (New in 0.6a1)
    하나 이상의 커스텀 "site" 디렉토리를 지정한다. "site" 디렉토리는 메인 파이썬 ``site-packages`` 디렉토리와 같이 ``.pth`` 파일이 처리되는 디렉토리이다. 
    0.6a10부터 EasyInstall은 주어진 디렉토리가 ``.pth`` 파일을 처리하는지의 여부에 대해 자동으로 감지한다.
    그래서 일반적으로 이 옵션을 사용할 필요는 없다. EasyInstall의 판단을 오버라이드하고 설치 디렉토리가 ``.pth`` 파일을 지원하는 것처럼 처리하기를 원한다면 필요하다. 
    
``--no-deps, -N``  (New in 0.6a6)
    종속성을 설치하지 마라. 이는 플랫폼 지정 패키지 시스템에서 에그를 래핑하는 도구의 편리성을 위한 것이다. 
    (다른 용도로 사용하지 않는 것이 좋다.)
    
``--allow-hosts=PATTERNS, -H PATTERNS``   (New in 0.6a6)
    지정된 glob 패턴과 일치하는 호스트에 대한 다운로드와 스파이더링을 제한한다. 예를 들면, ``-H *.python.org`` 은 웹 액세스에 대한 제한을 한다. 그래서 단지 패키지는 ``python.org`` 도메인에 있는 머신으로부터 나열되고 다운로드 할 수 있는 패키지를 제공한다. 
    glob 패턴은 대상 URL의 전체 사용자/호스트/포트 섹션과 일치해야한다. 예를 들어 ``* .python.org`` 는 ``http://python.org/foo`` 또는 ``http://www.python.org:8080/`` 과 같은 URL을 허용하지 않는다. 
    쉼표로 구분하여 여러 패턴을 지정할 수 있다. 기본 패턴은 무엇이든 일치하는 ``*`` 이다.

    일반적으로이 옵션은 EasyInstall의 웹 액세스를 완전히 차단 (예 : ``-Hlocalhost`` )하거나 인트라넷 또는 다른 신뢰할 수있는 사이트로 제한하는 데 주로 유용하다. EasyInstall은 호스트 제한 사항에 따라 종속성을 만족시킬 수있는 최선의 방법을 사용하지만 적절한 패키지를 찾지 못하면 실패 할 수도 있다.
    EasyInstall은 차단된 모든 URL을 표시하므로 의도보다 더 엄격한 경우,  `--allow-hosts` 설정을 조정할 수 있다.
    어떤 사이트에서는 `configuration files`_ 에 이 옵션에 대한 제한적인 기본 설정을 정의하고 필요에 따라 수동으로 설정을 덮어 쓸 수 있게 한다.


``--prefix=DIR`` (New in 0.6a10)
    지정된 디렉토리를 기본 설치 및 스크립트 디렉토리 계산을 위한 기반으로 사용해라. Windows의 경우, 기본 디렉토리는 ``prefix\\Lib\\site-packages`` 와 ``prefix\\Scripts`` 이다. 다른 플랫폼에서는 라이브러리에 대해서는 ``prefix/lib/python2.X/site-packages`` (적절한 버전으로 대체)를,  스크립트에 대해서는 ``prefix/bin`` 를 사용한다. 
    
    ``--prefix`` 옵션은 단지 기본 설치와 스크립트 디렉토리만 설정하고 커맨드 라인이나 구성 파일에 설정된 디렉토리를 덮어 쓰지 않는다는 것을 주의해라.


``--local-snapshots-ok, -l`` (New in 0.6c6)
    일반적으로 EasyInstall은 현재 릴리스 된 버전의 프로젝트만 설치하는 것을 선호한다. 개발 중인 프로젝트는 현재 유효한 버전 번호가 없을 수 있기 때문에 EasyInstall은 해당 프로젝트를 설치하는 것을 선호한다. 그래서 보통 ``setup.py`` 디렉토리가 커맨드라인에서 명시적으로 전달될 때만 설치한다.

    그러나, 이 옵션을 사용하면, ``setup.py develop`` 명령을 사용해서 설치된 개발 중인 프로젝트는 에그를 빌드할 것이다. 그리고 효율적으로 개발중인 프로젝트를 스냅샷 릴리즈로 효과적으로 업그레이드 할 것이다.

    일반적으로, 이 옵션은 응용 프로그램을 실행하는 데 필요한 모든 에그의 배포 가능한 스냅샷을 만들기 위해 ``--always-copy`` 옵션과 결합해서 사용된다. 

    이 옵션을 사용한다면, 개발 중인 프로젝트의 유효한 버전 넘버(예 : SVN 수정 번호 태그)가 있음을 확실히 해야한다. 그렇지 않으면 EasyInstall은 향후에 설치 또는 업그레이드를 시도할 때 어떤 프로젝트의 버전이 더 새로운 버전인지를 구별할 수 없다. 
    

.. _non-root installation:

커스텀 설치 위치 
-----------------------------

기본적으로 EasyInstall은 파이썬의 주요 ``site-packages`` 디렉토리에 파이썬 패키지를 설치하고 같은 디렉토리에있는 사용자 정의 ``.pth`` 파일을 사용하여 관리한다.

그러나 사용자 또는 개발자는 보통 easy_install을 설치하고 다른 위치에 파이썬 패키지를 설치하기를 원한다. 보통 3가지 이유 중 하나가 있다.:

1. 파이썬 메인 사이트 패키지 디렉토리에 쓸 수있는 권한이 없다.

2. 다른 사용자가 볼 수없는 패키지에 대한 사용자 고유의 은닉을 원한다.

3. 한 세트의 패키지를 특정 파이썬 응용 프로그램으로 격리하려고 한다. 보통 버전 충돌의 가능성을 최소화하기 위해서이다.

역사적으로 사용자 정의 설치를 수행하는 데는 여러 가지 방법이 있었다. 다음 섹션에서는 가장 쉽고 가장 관련있는 접근 방법을 나열한다.  [1]_.

`"--user" 옵션 사용`_

`"--user" 옵션 사용 및 "PYTHONUSERBASE" 커스터마이징`_

`"virtualenv" 사용`_

.. [1] ``easy_install` 과 ``setup.py install`` 옵션을 ``PYTHONPATH`` 및 ``PYTHONUSERBASE`` 변경과 결합하여 커스텀 설치를 하는 오래된 방법이 있다. 그러나 파이썬2.6에서 `PEP-370`_ 에 의해 가져온 사용자 체계에 의해 이들 모두는 추천되지 않는다. 

.. _PEP-370: http://www.python.org/dev/peps/pep-0370/


"--user" 옵션 사용
~~~~~~~~~~~~~~~~~~~~~~~
파이썬 2.6에서는 설치를 위한 사용자 스키마가 제공되었는데, 이는 모든 파이썬 배포판이 사용자에게 특정한 대체 설치 위치를 지원한다는 것을 의미한다 [2]_ [3]_.
각 OS의 기본 위치는 ``site.USER_BASE`` 변수에 대한 파이썬 문서에 설명되어 있다. 
이 설치 모드는 ``setup.py install`` 또는 ``easy_install`` 에 ``--user`` 옵션을 지정함으로써 가능하다. 
이 접근법은 패키지에 대한 사용자 고유의 은닉을 필요로 한다. 


.. [2] 파이썬 2.6 이전에는 Mac OS X이 사용자 스키마의 한 형태를 제공했다. 지금은 Python 2.6에서 소개된 사용자 체계에 포함되었다.
.. [3] 사용자 스키마 이전에는 여전히 사용할 수있는 홈 스키마가 있었지만 패키지를 인식하기 위해 사용자 스키마보다 많은 노력이 필요했다.

"--user" 옵션 사용 및 "PYTHONUSERBASE" 커스터마이징
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
사용자 스키마 설치 위치는 ``site.USER_BASE`` 값을 업데이트하는 ``PYTHONUSERBASE``  환경 변수를 설정하여 커스터마이징 할 수 있다. 특정 애플리케이션에 패키지를 격리하려면 해당 응용 프로그램의 OS 환경을 해당 패키지만 포함된 ``PYTHONUSERBASE`` 의 특정 값으로 설정하기만 하면 된다. 

"virtualenv" 사용
~~~~~~~~~~~~~~~~~~~
"virtualenv"는 파이썬 설치를 효과적으로 복제하는 써드파티 파이썬 패키지이고 이로 인해 설치 패키지에 고립된 위치를 만든다.
"virtualenv"의 발전은 사용자 설치 계획이 있기 전에 시작되었다.
"virtualenv"는 복제된 파이썬 설치에 범위내에 있고 정상적인 방법으로 사용되는 ``easy_install`` 버전을 제공한다.
"virtualenv"는 사용자 설치 스키마가 제공하지 않는 다양한 기능을 제공합니다 (예 : 주요 파이썬 사이트 패키지를 숨길 수 있는 기능.)


더 많은 정보를 위해서는 `virtualenv`_ 문서를 봐라.

.. _virtualenv: https://pypi.python.org/pypi/virtualenv



패키지 인덱스 "API"
-------------------

커스텀 패키지 인덱스(및 PyPI)은 패키지를 찾고 다운로드 할 수 있도록 EasyInstall에 대한 다음 규칙을 따라야 한다
.:

1. 달리 명시되지 않은 한, 페이지는 HTML 또는 XHTML이고 링크는 href의 속성을 참조한다.

2. 개별 프로젝트 버전 페이지의 URL은 ``base/projectname/versin`` 형식이어야한다. 여기서 ``base`` 는 패키지 인덱스의 기본 URL 이다.

3. 프로젝트 페이지의 URL의 ``/ version`` 부분을 생략하는 것은 (그러나 ``/`` 의 후행을 유지하면) 페이지가 생성의 결과가 된다.:

   a) 버전이 명시적으로 포함된 것처럼 해당 프로젝트의 단일 활성화 버전.

   b) 해당 프로젝트의 모든 활성화  버전 페이지에 대한 링크가있는 페이지.

4. 개별 프로젝트 버전 페이지는 다운로드 가능한 배포판에 대한 직접 링크를 포함해야한다. 프로젝트의 "long_description"이 URL을 포함하기 위해 명시적으로 허용되고 EasyInstall은 특정 페이지의 특정 부분이 인덱스와 관련된 것인지 또는 어떤 부분이 프로젝트가 제공한 설명인지를 확인하기 위해 특별한 처리를 하지 않으므로 패키지 인덱스에 의해 HTML 링크로 포맷이 바뀌어져야 한다. 

5. Where available, MD5 information should be added to download URLs by
   appending a fragment identifier of the form ``#md5=...``, where ``...`` is
   the 32-character hex MD5 digest.  EasyInstall will verify that the
   downloaded file's MD5 digest matches the given value.

6. 개별 프로젝트 버전 페이지는 해당 URL에 링크된 HTML 요소에서 ``rel="homepage"`` 과 ``rel="download"`` 을 사용해서 홈페이지 또는 다운로드를 식별해야한다. 이러한 속성은 검사로 다운로드 가능한 배포판인지 여부를 판단할 수 없는 경우 , EasyInstall이 항상 제공된 링크를 따르게 한다. 링크가 다운로드 가능한 배포판이 아니라면, 검색되고, HTML인 경우에는 다운로드 링크가 있는지 검색된다. 패키지 인덱스 사이트의 일부인 페이지에 대해서만 처리되므로 추가 홈페이지 또는 다운로드 링크를 검색하지 않는다. 


7. 후행 ``/`` 으로 검색된 인덱스의 루트 URL은 모든 프로젝트의 활성화 버전 페이지에 대한 링크가 포함된 페이지여야 한다.

   (노트: 이 요구사항은 URL 경로에서 프로젝트 이름의 대소문자를 구분하지 않는 ``safe_name()`` 이 없는 경우에 대한 방법이다. 프로젝트 이름이 (PyPI server, mod_rewriter 또는 유사한 메커니즘을 통해) 이 방식으로 일치한다면, 목록 페이지에 이 모든 패키지를 포함할 필요는 없다.

8. 패키지 인덱스가 ``file : //`` URL을 통해 접근된다면, URL에 ``/`` 를 포함한 디렉토리를 읽으려고 시도할 때, EasyInstall은  ``index.html`` 파일이 존재한다면 이를 사용할 것이다. 



이전 버전과의 호환성
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

0.6b4 이전의 setuptools 버전을 지원하기를 원하는 패키지 인덱스는 다음의 규칙을 따라야 한다.:

* 홈페이지와 다운로드 링크는 실제 링크에 ``rel = ""`` 속성에 덧붙여 (또는 대신에) ``<th> 홈 페이지`` 또는 ``<th> 다운로드 URL``` 로 시작해야한다. 그러나 이러한 마커 문자열은 표시될 필요가 없다. 예를 들면 다음은 setuptools의 어떤 버전에서도 작동하는 유효한 홈페이지 링크이다.::

    <li>
     <strong>Home Page:</strong>
     <!-- <th>Home Page -->
     <a rel="homepage" href="http://sqlobject.org">http://sqlobject.org</a>
    </li>

  마커 문자열이 HTML 주석이라고 해도, EasyInstall의 이전 버전은 여전히 그것을 확인할 것이고 다음에 오는 링크가 프로젝트의 홈페이지 URL임을 알 수 있다.
  
* 앞선 섹션의 문단 3(b)에서 설명된 페이지는 텍스트 어딘가에 반드시 ``"Index of Packages</title>"`` 문자열을 포함해야한다. 바람직하게는 HTML 주석내에 위치할 것이고 페이지 어느 곳에나 있을 수 있다. (노트: 문단 2와 3(a)에서 설명한대로, 이 문자열은 일반적인 프로젝트 페이지에 나타나면 안된다!)

추가적으로 ``#md5=`` fragnant ID를 사용하지 않는 PyPI 버전과의 호환성을 위해, EasyInstall은 PyPI의 표시된 MD5 정보(가독성을 위해 두 줄로 구분 시킨)를 일치시키기 위해 다음의 정규 표현식을 사용한다.::

    <a href="([^"#]+)">([^<]+)</a>\n\s+\(<a href="[^?]+\?:action=show_md5
    &amp;digest=([0-9a-f]{32})">md5</a>\)

History
=======

0.6c9
 * Fixed ``win32.exe`` support for .pth files, so unnecessary directory nesting
   is flattened out in the resulting egg.  (There was a case-sensitivity
   problem that affected some distributions, notably ``pywin32``.)

 * Prevent ``--help-commands`` and other junk from showing under Python 2.5
   when running ``easy_install --help``.

 * Fixed GUI scripts sometimes not executing on Windows

 * Fixed not picking up dependency links from recursive dependencies.

 * Only make ``.py``, ``.dll`` and ``.so`` files executable when unpacking eggs

 * Changes for Jython compatibility

 * Improved error message when a requirement is also a directory name, but the
   specified directory is not a source package.

 * Fixed ``--allow-hosts`` option blocking ``file:`` URLs

 * Fixed HTTP SVN detection failing when the page title included a project
   name (e.g. on SourceForge-hosted SVN)

 * Fix Jython script installation to handle ``#!`` lines better when
   ``sys.executable`` is a script.

 * Removed use of deprecated ``md5`` module if ``hashlib`` is available

 * Keep site directories (e.g. ``site-packages``) from being included in
   ``.pth`` files.

0.6c7
 * ``ftp:`` download URLs now work correctly.

 * The default ``--index-url`` is now ``https://pypi.python.org/simple``, to use
   the Python Package Index's new simpler (and faster!) REST API.

0.6c6
 * EasyInstall no longer aborts the installation process if a URL it wants to
   retrieve can't be downloaded, unless the URL is an actual package download.
   Instead, it issues a warning and tries to keep going.

 * Fixed distutils-style scripts originally built on Windows having their line
   endings doubled when installed on any platform.

 * Added ``--local-snapshots-ok`` flag, to allow building eggs from projects
   installed using ``setup.py develop``.

 * Fixed not HTML-decoding URLs scraped from web pages

0.6c5
 * Fixed ``.dll`` files on Cygwin not having executable permissions when an egg
   is installed unzipped.

0.6c4
 * Added support for HTTP "Basic" authentication using ``http://user:pass@host``
   URLs.  If a password-protected page contains links to the same host (and
   protocol), those links will inherit the credentials used to access the
   original page.

 * Removed all special support for Sourceforge mirrors, as Sourceforge's
   mirror system now works well for non-browser downloads.

 * Fixed not recognizing ``win32.exe`` installers that included a custom
   bitmap.

 * Fixed not allowing ``os.open()`` of paths outside the sandbox, even if they
   are opened read-only (e.g. reading ``/dev/urandom`` for random numbers, as
   is done by ``os.urandom()`` on some platforms).

 * Fixed a problem with ``.pth`` testing on Windows when ``sys.executable``
   has a space in it (e.g., the user installed Python to a ``Program Files``
   directory).

0.6c3
 * You can once again use "python -m easy_install" with Python 2.4 and above.

 * Python 2.5 compatibility fixes added.

0.6c2
 * Windows script wrappers now support quoted arguments and arguments
   containing spaces.  (Patch contributed by Jim Fulton.)

 * The ``ez_setup.py`` script now actually works when you put a setuptools
   ``.egg`` alongside it for bootstrapping an offline machine.

 * A writable installation directory on ``sys.path`` is no longer required to
   download and extract a source distribution using ``--editable``.

 * Generated scripts now use ``-x`` on the ``#!`` line when ``sys.executable``
   contains non-ASCII characters, to prevent deprecation warnings about an
   unspecified encoding when the script is run.

0.6c1
 * EasyInstall now includes setuptools version information in the
   ``User-Agent`` string sent to websites it visits.

0.6b4
 * Fix creating Python wrappers for non-Python scripts

 * Fix ``ftp://`` directory listing URLs from causing a crash when used in the
   "Home page" or "Download URL" slots on PyPI.

 * Fix ``sys.path_importer_cache`` not being updated when an existing zipfile
   or directory is deleted/overwritten.

 * Fix not recognizing HTML 404 pages from package indexes.

 * Allow ``file://`` URLs to be used as a package index.  URLs that refer to
   directories will use an internally-generated directory listing if there is
   no ``index.html`` file in the directory.

 * Allow external links in a package index to be specified using
   ``rel="homepage"`` or ``rel="download"``, without needing the old
   PyPI-specific visible markup.

 * Suppressed warning message about possibly-misspelled project name, if an egg
   or link for that project name has already been seen.

0.6b3
 * Fix local ``--find-links`` eggs not being copied except with
   ``--always-copy``.

 * Fix sometimes not detecting local packages installed outside of "site"
   directories.

 * Fix mysterious errors during initial ``setuptools`` install, caused by
   ``ez_setup`` trying to run ``easy_install`` twice, due to a code fallthru
   after deleting the egg from which it's running.

0.6b2
 * Don't install or update a ``site.py`` patch when installing to a
   ``PYTHONPATH`` directory with ``--multi-version``, unless an
   ``easy-install.pth`` file is already in use there.

 * Construct ``.pth`` file paths in such a way that installing an egg whose
   name begins with ``import`` doesn't cause a syntax error.

 * Fixed a bogus warning message that wasn't updated since the 0.5 versions.

0.6b1
 * Better ambiguity management: accept ``#egg`` name/version even if processing
   what appears to be a correctly-named distutils file, and ignore ``.egg``
   files with no ``-``, since valid Python ``.egg`` files always have a version
   number (but Scheme eggs often don't).

 * Support ``file://`` links to directories in ``--find-links``, so that
   easy_install can build packages from local source checkouts.

 * Added automatic retry for Sourceforge mirrors.  The new download process is
   to first just try dl.sourceforge.net, then randomly select mirror IPs and
   remove ones that fail, until something works.  The removed IPs stay removed
   for the remainder of the run.

 * Ignore bdist_dumb distributions when looking at download URLs.

0.6a11
 * Process ``dependency_links.txt`` if found in a distribution, by adding the
   URLs to the list for scanning.

 * Use relative paths in ``.pth`` files when eggs are being installed to the
   same directory as the ``.pth`` file.  This maximizes portability of the
   target directory when building applications that contain eggs.

 * Added ``easy_install-N.N`` script(s) for convenience when using multiple
   Python versions.

 * Added automatic handling of installation conflicts.  Eggs are now shifted to
   the front of sys.path, in an order consistent with where they came from,
   making EasyInstall seamlessly co-operate with system package managers.

   The ``--delete-conflicting`` and ``--ignore-conflicts-at-my-risk`` options
   are now no longer necessary, and will generate warnings at the end of a
   run if you use them.

 * Don't recursively traverse subdirectories given to ``--find-links``.

0.6a10
 * Added exhaustive testing of the install directory, including a spawn test
   for ``.pth`` file support, and directory writability/existence checks.  This
   should virtually eliminate the need to set or configure ``--site-dirs``.

 * Added ``--prefix`` option for more do-what-I-mean-ishness in the absence of
   RTFM-ing.  :)

 * Enhanced ``PYTHONPATH`` support so that you don't have to put any eggs on it
   manually to make it work.  ``--multi-version`` is no longer a silent
   default; you must explicitly use it if installing to a non-PYTHONPATH,
   non-"site" directory.

 * Expand ``$variables`` used in the ``--site-dirs``, ``--build-directory``,
   ``--install-dir``, and ``--script-dir`` options, whether on the command line
   or in configuration files.

 * Improved SourceForge mirror processing to work faster and be less affected
   by transient HTML changes made by SourceForge.

 * PyPI searches now use the exact spelling of requirements specified on the
   command line or in a project's ``install_requires``.  Previously, a
   normalized form of the name was used, which could lead to unnecessary
   full-index searches when a project's name had an underscore (``_``) in it.

 * EasyInstall can now download bare ``.py`` files and wrap them in an egg,
   as long as you include an ``#egg=name-version`` suffix on the URL, or if
   the ``.py`` file is listed as the "Download URL" on the project's PyPI page.
   This allows third parties to "package" trivial Python modules just by
   linking to them (e.g. from within their own PyPI page or download links
   page).

 * The ``--always-copy`` option now skips "system" and "development" eggs since
   they can't be reliably copied.  Note that this may cause EasyInstall to
   choose an older version of a package than what you expected, or it may cause
   downloading and installation of a fresh version of what's already installed.

 * The ``--find-links`` option previously scanned all supplied URLs and
   directories as early as possible, but now only directories and direct
   archive links are scanned immediately.  URLs are not retrieved unless a
   package search was already going to go online due to a package not being
   available locally, or due to the use of the ``--update`` or ``-U`` option.

 * Fixed the annoying ``--help-commands`` wart.

0.6a9
 * Fixed ``.pth`` file processing picking up nested eggs (i.e. ones inside
   "baskets") when they weren't explicitly listed in the ``.pth`` file.

 * If more than one URL appears to describe the exact same distribution, prefer
   the shortest one.  This helps to avoid "table of contents" CGI URLs like the
   ones on effbot.org.

 * Quote arguments to python.exe (including python's path) to avoid problems
   when Python (or a script) is installed in a directory whose name contains
   spaces on Windows.

 * Support full roundtrip translation of eggs to and from ``bdist_wininst``
   format.  Running ``bdist_wininst`` on a setuptools-based package wraps the
   egg in an .exe that will safely install it as an egg (i.e., with metadata
   and entry-point wrapper scripts), and ``easy_install`` can turn the .exe
   back into an ``.egg`` file or directory and install it as such.

0.6a8
 * Update for changed SourceForge mirror format

 * Fixed not installing dependencies for some packages fetched via Subversion

 * Fixed dependency installation with ``--always-copy`` not using the same
   dependency resolution procedure as other operations.

 * Fixed not fully removing temporary directories on Windows, if a Subversion
   checkout left read-only files behind

 * Fixed some problems building extensions when Pyrex was installed, especially
   with Python 2.4 and/or packages using SWIG.

0.6a7
 * Fixed not being able to install Windows script wrappers using Python 2.3

0.6a6
 * Added support for "traditional" PYTHONPATH-based non-root installation, and
   also the convenient ``virtual-python.py`` script, based on a contribution
   by Ian Bicking.  The setuptools egg now contains a hacked ``site`` module
   that makes the PYTHONPATH-based approach work with .pth files, so that you
   can get the full EasyInstall feature set on such installations.

 * Added ``--no-deps`` and ``--allow-hosts`` options.

 * Improved Windows ``.exe`` script wrappers so that the script can have the
   same name as a module without confusing Python.

 * Changed dependency processing so that it's breadth-first, allowing a
   depender's preferences to override those of a dependee, to prevent conflicts
   when a lower version is acceptable to the dependee, but not the depender.
   Also, ensure that currently installed/selected packages aren't given
   precedence over ones desired by a package being installed, which could
   cause conflict errors.

0.6a3
 * Improved error message when trying to use old ways of running
   ``easy_install``.  Removed the ability to run via ``python -m`` or by
   running ``easy_install.py``; ``easy_install`` is the command to run on all
   supported platforms.

 * Improved wrapper script generation and runtime initialization so that a
   VersionConflict doesn't occur if you later install a competing version of a
   needed package as the default version of that package.

 * Fixed a problem parsing version numbers in ``#egg=`` links.

0.6a2
 * EasyInstall can now install "console_scripts" defined by packages that use
   ``setuptools`` and define appropriate entry points.  On Windows, console
   scripts get an ``.exe`` wrapper so you can just type their name.  On other
   platforms, the scripts are installed without a file extension.

 * Using ``python -m easy_install`` or running ``easy_install.py`` is now
   DEPRECATED, since an ``easy_install`` wrapper is now available on all
   platforms.

0.6a1
 * EasyInstall now does MD5 validation of downloads from PyPI, or from any link
   that has an "#md5=..." trailer with a 32-digit lowercase hex md5 digest.

 * EasyInstall now handles symlinks in target directories by removing the link,
   rather than attempting to overwrite the link's destination.  This makes it
   easier to set up an alternate Python "home" directory (as described above in
   the `Non-Root Installation`_ section).

 * Added support for handling MacOS platform information in ``.egg`` filenames,
   based on a contribution by Kevin Dangoor.  You may wish to delete and
   reinstall any eggs whose filename includes "darwin" and "Power_Macintosh",
   because the format for this platform information has changed so that minor
   OS X upgrades (such as 10.4.1 to 10.4.2) do not cause eggs built with a
   previous OS version to become obsolete.

 * easy_install's dependency processing algorithms have changed.  When using
   ``--always-copy``, it now ensures that dependencies are copied too.  When
   not using ``--always-copy``, it tries to use a single resolution loop,
   rather than recursing.

 * Fixed installing extra ``.pyc`` or ``.pyo`` files for scripts with ``.py``
   extensions.

 * Added ``--site-dirs`` option to allow adding custom "site" directories.
   Made ``easy-install.pth`` work in platform-specific alternate site
   directories (e.g. ``~/Library/Python/2.x/site-packages`` on Mac OS X).

 * If you manually delete the current version of a package, the next run of
   EasyInstall against the target directory will now remove the stray entry
   from the ``easy-install.pth`` file.

 * EasyInstall now recognizes URLs with a ``#egg=project_name`` fragment ID
   as pointing to the named project's source checkout.  Such URLs have a lower
   match precedence than any other kind of distribution, so they'll only be
   used if they have a higher version number than any other available
   distribution, or if you use the ``--editable`` option.  The ``#egg``
   fragment can contain a version if it's formatted as ``#egg=proj-ver``,
   where ``proj`` is the project name, and ``ver`` is the version number.  You
   *must* use the format for these values that the ``bdist_egg`` command uses;
   i.e., all non-alphanumeric runs must be condensed to single underscore
   characters.

 * Added the ``--editable`` option; see `소스 패키지 수정 및 보기`_
   above for more info.  Also, slightly changed the behavior of the
   ``--build-directory`` option.

 * Fixed the setup script sandbox facility not recognizing certain paths as
   valid on case-insensitive platforms.

0.5a12
 * Fix ``python -m easy_install`` not working due to setuptools being installed
   as a zipfile.  Update safety scanner to check for modules that might be used
   as ``python -m`` scripts.

 * Misc. fixes for win32.exe support, including changes to support Python 2.4's
   changed ``bdist_wininst`` format.

0.5a10
 * Put the ``easy_install`` module back in as a module, as it's needed for
   ``python -m`` to run it!

 * Allow ``--find-links/-f`` to accept local directories or filenames as well
   as URLs.

0.5a9
 * EasyInstall now automatically detects when an "unmanaged" package or
   module is going to be on ``sys.path`` ahead of a package you're installing,
   thereby preventing the newer version from being imported.  By default, it
   will abort installation to alert you of the problem, but there are also
   new options (``--delete-conflicting`` and ``--ignore-conflicts-at-my-risk``)
   available to change the default behavior.  (Note: this new feature doesn't
   take effect for egg files that were built with older ``setuptools``
   versions, because they lack the new metadata file required to implement it.)

 * The ``easy_install`` distutils command now uses ``DistutilsError`` as its
   base error type for errors that should just issue a message to stderr and
   exit the program without a traceback.

 * EasyInstall can now be given a path to a directory containing a setup
   script, and it will attempt to build and install the package there.

 * EasyInstall now performs a safety analysis on module contents to determine
   whether a package is likely to run in zipped form, and displays
   information about what modules may be doing introspection that would break
   when running as a zipfile.

 * Added the ``--always-unzip/-Z`` option, to force unzipping of packages that
   would ordinarily be considered safe to unzip, and changed the meaning of
   ``--zip-ok/-z`` to "always leave everything zipped".

0.5a8
 * There is now a separate documentation page for `setuptools`_; revision
   history that's not specific to EasyInstall has been moved to that page.

 .. _setuptools: http://peak.telecommunity.com/DevCenter/setuptools

0.5a5
 * Made ``easy_install`` a standard ``setuptools`` command, moving it from
   the ``easy_install`` module to ``setuptools.command.easy_install``.  Note
   that if you were importing or extending it, you must now change your imports
   accordingly.  ``easy_install.py`` is still installed as a script, but not as
   a module.

0.5a4
 * Added ``--always-copy/-a`` option to always copy needed packages to the
   installation directory, even if they're already present elsewhere on
   sys.path. (In previous versions, this was the default behavior, but now
   you must request it.)

 * Added ``--upgrade/-U`` option to force checking PyPI for latest available
   version(s) of all packages requested by name and version, even if a matching
   version is available locally.

 * Added automatic installation of dependencies declared by a distribution
   being installed.  These dependencies must be listed in the distribution's
   ``EGG-INFO`` directory, so the distribution has to have declared its
   dependencies by using setuptools.  If a package has requirements it didn't
   declare, you'll still have to deal with them yourself.  (E.g., by asking
   EasyInstall to find and install them.)

 * Added the ``--record`` option to ``easy_install`` for the benefit of tools
   that run ``setup.py install --record=filename`` on behalf of another
   packaging system.)

0.5a3
 * Fixed not setting script permissions to allow execution.

 * Improved sandboxing so that setup scripts that want a temporary directory
   (e.g. pychecker) can still run in the sandbox.

0.5a2
 * Fix stupid stupid refactoring-at-the-last-minute typos.  :(

0.5a1
 * Added support for converting ``.win32.exe`` installers to eggs on the fly.
   EasyInstall will now recognize such files by name and install them.

 * Fixed a problem with picking the "best" version to install (versions were
   being sorted as strings, rather than as parsed values)

0.4a4
 * Added support for the distutils "verbose/quiet" and "dry-run" options, as
   well as the "optimize" flag.

 * Support downloading packages that were uploaded to PyPI (by scanning all
   links on package pages, not just the homepage/download links).

0.4a3
 * Add progress messages to the search/download process so that you can tell
   what URLs it's reading to find download links.  (Hopefully, this will help
   people report out-of-date and broken links to package authors, and to tell
   when they've asked for a package that doesn't exist.)

0.4a2
 * Added support for installing scripts

 * Added support for setting options via distutils configuration files, and
   using distutils' default options as a basis for EasyInstall's defaults.

 * Renamed ``--scan-url/-s`` to ``--find-links/-f`` to free up ``-s`` for the
   script installation directory option.

 * Use ``urllib2`` instead of ``urllib``, to allow use of ``https:`` URLs if
   Python includes SSL support.

0.4a1
 * Added ``--scan-url`` and ``--index-url`` options, to scan download pages
   and search PyPI for needed packages.

0.3a4
 * Restrict ``--build-directory=DIR/-b DIR`` option to only be used with single
   URL installs, to avoid running the wrong setup.py.

0.3a3
 * Added ``--build-directory=DIR/-b DIR`` option.

 * Added "installation report" that explains how to use 'require()' when doing
   a multiversion install or alternate installation directory.

 * Added SourceForge mirror auto-select (Contributed by Ian Bicking)

 * Added "sandboxing" that stops a setup script from running if it attempts to
   write to the filesystem outside of the build area

 * Added more workarounds for packages with quirky ``install_data`` hacks

0.3a2
 * Added subversion download support for ``svn:`` and ``svn+`` URLs, as well as
   automatic recognition of HTTP subversion URLs (Contributed by Ian Bicking)

 * Misc. bug fixes

0.3a1
 * Initial release.


Future Plans
============

* Additional utilities to list/remove/verify packages
* Signature checking?  SSL?  Ability to suppress PyPI search?
* Display byte progress meter when downloading distributions and long pages?
* Redirect stdout/stderr to log during run_setup?


==================================================
Building and Distributing Packages with Setuptools
==================================================

``Setuptools`` 는 개발자가 Python 패키지, 특히 다른 패키지에 dependency가 있는 패키지를 보다
쉽게 ​​빌드하고 배포 할 수 있게 해주는 Python 2.6 이상의 ``distutils`` 의 향상된 기능 모음이다.

``setuptools`` 를 사용하여 빌드되고 배포되는 패키지는 사용자에게는 ``distutils`` 에 기반한 일반적인
Python 패키지처럼 보이게 된다. 사용자는 setuptools를 설치하거나 사용할 필요가 없으므로 배포판에
setuptools 패키지를 포함 할 필요가 없다. 하나의 `bootstrap module`_ (12K .py 파일)을
포함시키면 사용자가 패키지를 소스에서 빌드할 시, 적절한 버전이 이미 설치되어 있지 않으면 패키지는 자동으로
``setuptools`` 를 다운로드하여 설치하게 된다.

.. _bootstrap module: https://bootstrap.pypa.io/ez_setup.py

주요 기능:

* `EasyInstall tool <easy_install.html>`_ 를 사용하여 빌드 단계에서 자동으로 dependency를
  검색/다운로드/설치/업그레이드. HTTP, FTP, Subversion, SourceForge를 지원하며 PyPI에서
  링크된 웹페이지들을 자동으로 스캔하여 다운로드 링크를 찾는다. 이는 현재 Python에서 CPAN에 가장
  가까운 물건이다.

* 단일 파일 import가능한 distribution format인 `Python Eggs
  <http://peak.telecommunity.com/DevCenter/PythonEggs>`_ 를 생성.

* zip 패키지로 호스트 된 데이터 파일에 대한 액세스 향상된 지원.

* 소스 트리에 있는 모든 패키지를 setup.py에 개별적으로 나열하지 않고 자동으로 포함.

* ``MANIFEST.in`` 파일을 생성 할 필요 없이, 그리고 소스 트리가 변경 될 때 ``MANIFEST`` 파일을
  강제로 재생성하지 않고 소스 배포판에 모든 관련 파일을 자동으로 포함.

* 자동으로 프로젝트에서 "주요" 기능의 대한 wrapper 스크립트 또는 콘솔 및 GUI Windows .exe 파일을
  생성. (주의: 이것은 py2exe 대체 파일이 아니며 .exe 파일은 로컬 Python 설치에 의존한다.)

* setup.py가 ``.pyx`` 파일을 나열 할 수 있고 최종 사용자가 Pyrex를 설치하지 않은 경우에도
  (Pyrex가 생성 한 C를 소스 배포판에 포함시키는 한) 작동하도록 투명한 Pyrex 지원.

* Command alias 기능. 일반적으로 사용되는 command 및 옵션에 대한 프로젝트 별, 사용자 별, 사이트
  전체 바로 가기 이름을 만든다.

* PyPI 업로드 지원. 소스 배포판과 egg를 PyPI에 업로드

* ``development mode`` 로 프로젝트 배포. ``sys.path`` 에서 사용할 수 있지만 소스
  checkout에서도 직접 편집 할 수 있다.

* 새로운 command나 ``setup()`` argument로 distutils를 쉽게 확장 할 수 있으며, 코드를
  복사하지 않고도 여러 프로젝트에 extension을 배포/재사용.

* 프로젝트의 설치 스크립트에 선언 된 간단한 "entry point"를 사용하여 extension을 자동으로 발견하는
  확장 가능한 응용 프로그램 및 프레임워크를 생성.

.. contents:: **Table of Contents**

.. _ez_setup.py: `bootstrap module`_


-----------------
Developer's Guide
-----------------


Installing ``setuptools``
=========================

`EasyInstall Installation Instructions`_ 를 따라 setuptools의 현재 stable 버전을
설치한다. 특히, Python의 ``site-packages`` 디렉토리가 아닌 곳에 설치하는 경우,
`Custom Installation Locations`_ 섹션을 반드시 읽어본다.

.. _EasyInstall Installation Instructions: easy_install.html#installation-instructions

.. _Custom Installation Locations: easy_install.html#custom-installation-locations

현재 개발중인 버전의 setuptools를 사용하려면 먼저 stable 버전을 설치 한 다음 다음을 실행한다::

    ez_setup.py setuptools==dev

이렇게하면 Python Subversion sandbox에서 setuptools의 최신 개발 버전을 다운로드하여 설치하게
된다.


Basic Use
=========

setuptools의 기본 사용을 위해서는 distutils 대신에 setuptools에서 import한다. 다음은
setuptools를 사용하는 간단한 설치 스크립트이다::

    from setuptools import setup, find_packages
    setup(
        name="HelloWorld",
        version="0.1",
        packages=find_packages(),
    )

보다시피, 프로젝트에서 setuptools를 사용하는 것은 그리 어렵지 않다. 개발한 Python 패키지와 함께
스크립트를 프로젝트 폴더에서 실행하면 된다.

이 스크립트를 실행하면, egg를 생성하고, PyPI에 업로드하고, setup.py가 있는 디렉토리에 모든 패키지를
자동으로 포함시킨다. 이 설정 스크립트에 어떤 명령을 줄 수 있는지 알아 보려면 아래의
`Command Reference`_ 섹션을 참조한다. 예를 들어, 소스 배포판을 생성하려면 다음을 실행한다::

    python setup.py sdist

물론, 프로젝트를 PyPI에 공개하기 전에 설정 스크립트에 좀 더 많은 정보를 추가하여 사람들이 프로젝트를
찾거나 배우는 데 도움이 되길 원할 것이다. 그리고 어쩌면 프로젝트는 그때까지 몇가지 dependency, 데이터
파일과 스크립트를 추가로 포함하게 되었을지도 모른다::

    from setuptools import setup, find_packages
    setup(
        name="HelloWorld",
        version="0.1",
        packages=find_packages(),
        scripts=['say_hello.py'],

        # Project uses reStructuredText, so ensure that the docutils get
        # installed or upgraded on the target machine
        install_requires=['docutils>=0.3'],

        package_data={
            # If any package contains *.txt or *.rst files, include them:
            '': ['*.txt', '*.rst'],
            # And include any *.msg files found in the 'hello' package, too:
            'hello': ['*.msg'],
        },

        # metadata for upload to PyPI
        author="Me",
        author_email="me@example.com",
        description="This is an Example Package",
        license="PSF",
        keywords="hello world example examples",
        url="http://example.com/HelloWorld/",   # project home page, if any

        # could also include long_description, download_url, classifiers, etc.
    )

다음 섹션에서는 우리는 이러한 ``setup()`` 의 argument 대부분(메타 데이터를 제외하고)이 무엇을
하는지, 그리고 프로젝트에서 사용 할 수 있는 다양한 방법을 설명한다.


Specifying Your Project's Version
---------------------------------

Setuptools는 대부분의 버전 관리 체계에서 잘 작동 할 수 있다. 그러나 setuptools와 EasyInstall이
패키지의 어떤 버전이 다른 버전보다 새로운 버전인지 항상 확인할 수 있도록 몇 가지 특별한 사항을 주의해야
한다. 이러한 것들을 알면 프로젝트가 의존하는 다른 프로젝트들의 버전을 정확하게 지정하는데 도움이 된다.

버전은 release 번호와 pre-release 또는 post-release 태그 번갈아 가며 구성된다. release 번호는
``2.4`` 또는 ``0.5`` 와 같이 점으로 구분 된 일련의 숫자이다. 점들 사이의 단위는 숫자로 처리되므로,
``2.1`` 과 ``2.1.0`` 은 동일한 release 번호를 다르게 표기하는 방법일 뿐이다. 이는 release 2의
첫 번째 subrelease를 나타낸다. 그러나 ``2.10`` 은 release 2의 *10번째* subrelease이므로
``2.1`` 또는 ``2.1.0`` 과는 다른 더 새로운 버전의 release이다. 또한 선행 0은 무시되므로
``2.01`` 은 ``2.1`` 과 같고 ``2.0.1`` 과는 다르다.

Release 번호 다음에는 pre-release 또는 post-release 태그가 있을 수 있다. Pre-release 태그는
태그가 수식하는 버전보다 *오래된* 것으로 간주되도록 한다. 따라서, revision ``2.4`` 는 revision
``2.4c1`` 보다 새로운 것이며, 이것은 ``2.4b1`` 또는 ``2.4a1`` 보다 새로운 것이다.
Post-release 태그는 태그가 수식하는 버전보다 *새로운* 것으로 간주되도록 한다. 따라서, revision
``2.4-1``, ``2.4pl3`` 과 같은 revision은 ``2.4`` 보다는 새롭지만 ``2.4.1`` 보다는 더
오래된 버전이다.

Pre-release 태그는 사전적으로 "final" 앞에 오는 일련의 문자들이다. Pre-release 태그의 예로는
``alpha``, ``beta``, ``a``, ``c``, ``dev`` 등이 있다. Pre-release 태그 앞에 숫자가
있다면, 점이나 대시를 넣을 필요는 없다. 따라서 ``2.4c1`` 과 ``2.4.c1`` 과 ``2.4-c1`` 은 모두
``2.4`` 버전의 release candidate 1을 나타내며, setuptools에 의해 동일하게 취급된다.

또한, ``pre``, ``preview``, ``rc`` 는 pre-release 태그로 특별하게 ``c`` 와 동일하게
취급된다. 따라서 ``2.4rc1``, ``2.4pre1``, ``2.4preview1`` 은 ``2.4c1`` 과 완전히 똑같은
버전이며, setuptools에 의해 동일하게 취급된다.

Post-release 태그는 사전적 정렬에서 "final" 보다 뒤에 오는 일련의 문자이거나, 대시(``-``)가
붙는다. Post-release 태그는 일반적으로 release 번호에서 패치 번호, 포트 번호, 빌드 번호, 개정
번호, 날짜 스탬프를 분리하는 데 사용된다. 예를 들어서, ``2.4-r1263`` 버전은 ``2.4`` 의 release
후 패치의 Subversion revision 1263을 나타낸다. 또는 ``2.4-20051127`` 을 사용하여 날짜가 찍힌
post-release를 나타낼 수도 있다.

각 pre-release 또는 post-release 태그 다음에 다른 release 번호를 자유롭게 넣을 수 있으며,
여기에 다시 pre-release 또는 post-release 태그를 더 추가 할 수도 있다. 예를 들어
``0.6a9.dev-r41475`` 는 release 0.6의 9번째 알파 버전의 개발 버전인 Subversion
revision 41475를 나타낸다. ``dev`` 는 출시 전 태그이므로, 이 버전은 release 0.6의 9번째
알파 버전인 ``0.6a9`` 보다 *낮은* 버전이다. 그러나 ``-r41475`` 는 릴리스 이후 태그이므로,
이 버전은 ``0.6a9.dev`` 보다 더 *새로운* 버전이다.

대부분의 경우, setuptools의 버전 번호 해석은 직관적이지만, 다음과 같은 몇 가지 팁을
통해 헷갈리는 경우의 문제를 해결할 수 있다:

* Pre-release 태그 여러개를 사이에 숫자나 점 없이 인접하게 붙이면 안된다.
  버전 ``1.9adev`` 은 ``1.9`` 버전의 ``adev`` pre-release를 의미하며, ``1.9a`` 의
  개발 pre-release를 의미하지 *않는다*. ``1.9a.dev`` 처럼 ``.dev`` 를 이용하거나
  ``1.9a0dev`` 처럼 숫자를 이용하여 분리해야 한다. 이 경우, ``1.9a.dev``, ``1.9a0dev``,
  ``1.9.a.dev`` 는 setuptools에 의해 전부 동일하게 취급된다.

* 선택한 버전 번호 체계가 생각대로 작동하는지 확인하려면, ``pkg_resources.parse_version()``
  함수를 ​​사용하여 서로 다른 버전 번호를 비교하면 된다::

    >>> from pkg_resources import parse_version
    >>> parse_version('1.9.a.dev') == parse_version('1.9a0dev')
    True
    >>> parse_version('2.1-rc2') < parse_version('2.1')
    True
    >>> parse_version('0.6a9dev-r41475') < parse_version('0.6a9')
    True

프로젝트의 버전 번호 체계를 결정하면, setuptools가 개발중인 release들에 다양한 pre-release 태그
또는 post-release 태그를 자동으로 태그하도록 할 수 있다. 자세한 내용은 다음 섹션들을 참조:

* `Tagging and "Daily Build" or "Snapshot" Releases`_
* `Managing "Continuous Releases" Using Subversion`_
* The `egg_info`_ command


New and Changed ``setup()`` Keywords
====================================

``setup()`` 에 대한 다음의 키워드 argument는 ``setuptools`` 에 의해 추가되거나 변경되었다.
모두 선택 사항이며, 관련된 ``setuptools`` 기능을 필요로 하지 않는다면 제공 할 필요는 없다.

``include_package_data``
``True`` 로 설정된다면, ``setuptools`` 는 ``MANIFEST.in`` 파일에 지정된 패키지 디렉토리
안에 있는 모든 데이터 파일을 자동으로 포함시킨다. 자세한 내용은 아래의 `Including Data Files`_
섹션을 참조.

``exclude_package_data``
패키지 이름을 패키지 디렉토리에서 *제외* 되어야 할 glob 패턴 목록에 매핑하는 dictionary이다.
이것을 사용하여 ``include_package_data`` 에 불필요하게 포함 된 파일들을 줄일 수 있다.
설명과 예제는 아래의 `Including Data Files`_ 를 참조.

``package_data``
패키지 이름을 glob 패턴 목록에 매핑하는 dictionary. 설명과 예제는 아래의 `Including Data Files`_
를 참조. ``include_package_data`` 를 사용하고 있다면, 이 옵션은 사용할 필요가 없다.
다만 설정 스크립트와 빌드 과정에서 생성 된 파일을 추가 할 필요있다면 사용한다.

``zip_safe``
프로젝트를 zip 파일에서 안전하게 설치하고 실행할 수 있는지를 지정하는 bool 플래그. 이 argument가
제공되지 않으면, ``bdist_egg`` 명령은 egg를 빌드 할 때마다 발생할 수있는 문제에 대해 프로젝트의
모든 내용을 분석해야 한다.

``install_requires``
이 패키지를 설치하기 전에 설치해야 할 다른 distribution을 지정하는 string 또는 string 목록.
이 argument의 형식과 예제에 대한 자세한 내용은 아래의 `Declaring Dependencies`_ 를 참조.

``entry_points``
Entry point 그룹 이름을 entry point를 정의하는 string 또는 string 목록에 매핑하는 dictionary.
Entry point는 프로젝트에서 제공하는 서비스 또는 플러그인의 동적 발견을 지원하는 데 사용된다.
이 argument의 형식에 대한 자세한 내용과 예는 `Dynamic Discovery of Services and Plugins`_
를 참조. 이 키워드는 `Automatic Script Creation`_ 을 지원하기 위해 사용되기도 한다.

``extras_require``
"extras"(프로젝트의 선택적 기능) 이름을 이러한 기능을 지원하기 위해 사전에 설치해야하는
distribution을 지정하는 string이나 string 목록에 매핑하는 dictionary. 이 argument의 형식과
예제에 대한 자세한 내용은 `Declaring Dependencies`_ 를 참조.

``python_requires``
Python 버전의 PEP440에 정의된 버전 지정자에 해당하는 string. PEP 345에 정의 된
Requires-Python을 지정하는 데 사용된다.

``setup_requires``
*setup 스크립트* 가 실행되기 위해서 필요한 다른 distribution을 지정하는 string 또는 string 목록.
``setuptools`` 는 나머지 설치 스크립트나 명령을 처리하기 전에 이것들을 얻으려고 시도(``EasyInstall`` 을
사용하여 다운로드하는 등)한다. 이 argument는 빌드 프로세스의 일부로 distutils extension을 사용하는
경우 필요하다. 한 예로, setup() argument를 처리하고 이를 EGG-INFO 메타데이터 파일로 변환하는
extension이 있다.

    (주의: ``setup_requires`` 에 나열된 프로젝트는 설치 스크립트가 실행되는 시스템에 자동으로 설치되지
않는다. 이미 로컬에 있지 않은 경우 단순히 ./.eggs 디렉토리로 다운로드 될 뿐이다. 만약 이들이 설치되고
설치 스크립트가 실행될 때 사용 가능하길 원한다면 ``install_requires``, ``setup_requires`` 에 **함께**
추가해야 한다.)

``dependency_links``
dependency를 검색 할 때 검색 될 URL들을 지정하는 string 목록. 이러한 링크는 ``setup_requires``
또는 ``tests_require`` 로 지정된 패키지를 설치하는 데 필요할 경우 사용된다. 또한 EasyInstall과 같은
도구로 ``.egg`` 파일을 설치할 때 사용하기 위해 egg의 메타데이터에 기록된다.

``namespace_packages``
프로젝트의 "namespace package"를 지정하는 string 목록. Namespace package는 여러 프로젝트
distribution에 걸쳐 분할 될 수 있는 패키지를 말한다. 예를 들어 Zope 3의 ``zope`` 패키지는
``zope.interface`` 와 ``zope.publisher`` 와 같은 서브패키지가 따로 배포 될 수 있기 때문에,
namespace package이다. Egg 런타임 시스템은 이러한 서브패키지들을 런타임에서 자동으로
하나의 상위 패키지로 합칠 수 있다. 단, 이 경우 namespace package의 ``__init__. py`` 에는
namespace 선언 외의 코드가 있어서는 안된다. 더 자세한 정보는 `Namespace Packages`_ 를 참조.

``test_suite``
``unittest.TestCase`` subclass(또는 그런 subclass나 subclass의 method 하나 이상을 포함하는
package나 module)를 지정하는 string 또는 argument 없이 호출 할 경우 ``unittest.TestSuite`` 를
반환하는 function. 만약 지정된 suite가 module이고, module이 ``additional_tests()`` function을
​​가지고 있다면, 그 function은 호출되고 결과는 실행할 테스트에 추가된다. 만약 지정된 suite가 package라면,
모든 submodule과 subpackage가 전체 test suite에 재귀적으로 추가된다.

    이 argument를 지정하면 `test`_ command를 사용하여 지정된 test suite를 실행 할 수 있다. (예 :``setup.py test``)
자세한 내용은 아래의 `test`_ command를 참조.

``tests_require``
프로젝트의 테스트가 그것을 설치하는 데 필요한 것 외에 하나 이상의 추가 패키지를 필요로 한다면,
이 옵션을 사용하여 지정할 수 있다. 패키지 테스트를 실행하기 위해 다른 distribution이 있어야 하는지를
지정하는 string 또는 string 목록이어야 한다. ``test`` command를 실행하면, ``setuptools`` 는
이것들을 얻으려고 시도(``EasyInstall`` 을 사용하여 다운로드하는 것 포함)한다. 이 필수 프로젝트들은
테스트가 실행되는 시스템에 설치되지 않고, 로컬에 아직 설치되지 않은 경우에 프로젝트의 설치 디렉토리로
다운로드만 된다.

.. _test_loader:

``test_loader``
setuptools가 일반적으로 사용하는 것보다 실행할 테스트를 찾는 다른 방법을 사용하려면,
이 argument에 module 이름과 class 이름을 지정할 수 있다. 지정된 class는 argument 없이
인스턴스화 가능해야 하며, 인스턴스는 Python의 `unittest` module의 ``TestLoader`` class에 정의된
``loadTestsFromNames()`` moethod를 지원해야 한다. Setuptools는 `names` argument에
``test_suite`` argument에 제공된 값인 하나의 테스트 "name"만 전달한다. ``test_suite``
string에 포함가능 한 것에 제한이 없으므로, 지정한 로더는 원하는 대로 이 string을 해석 할 수 있다.

    Module 이름과 class 이름은 ``:`` 로 구분되어야 한다. 이 argument의 default값은
``"setuptools.command.test:ScanningLoader"`` 이다. Default인 ``unittest`` 동작을 사용하고자 한다면,
대신 ``"unittest:TestLoader"`` 를 ``test_loader`` argument로 지정할 수도 있다. 다만, 이렇게 하면
submodule 및 subpackage를 자동으로 검색 할 수 없다.

    여기에 지정한 module과 class는 다른 package에 포함 될 수도 있다. 다만 이 경우, ``tests`` command를
실행 할 때 loader class를 포함하는 package를 사용할 수 있도록 ``tests_require`` 옵션을 사용해야 한다.

``eager_resources``
함께 추출되어야 하는 리소스를 지정하는 string 목록. 이 argument는 프로젝트가 zip 파일로 설치되며,
목록의 모든 리소스가 *단위* 로 파일 시스템에 추출되어야 하는 경우에만 유용하다. 여기에 나열된 리소스는
소스 root에 상대적으로 경로가 '/'로 분리되어 있어야 한다. 따라서 패키지 ``bar.baz`` 에 리소스 ``foo.png`` 를
나열하려면 ``bar/baz/foo.png`` 를 사용한다.

    리소스를 한 번에 하나씩만 가져 오거나, 프로젝트의 다른 파일(데이터 파일 또는 공유 라이브러리)에 접근하는
C extension이 없으면, 이 인수가 필요하지 않은 겨우일 가능성이 높으므로 이 설정을 건드리지 않는게 좋다.
이 argument의 작동 방식에 대한 자세한 내용은 아래의 `Automatic Resource Extraction`_ 을 참조.

``use_2to3``
빌드 과정에서 Python 2에서 Python 3으로 소스 코드를 변환. 자세한 것은 :doc:`python3` 를 참조.

``convert_2to3_doctests``
2to3로 변환 될 필요가 있는 doctest 소스 파일 목록. 자세한 것은 :doc:`python3` 를 참조.

``use_2to3_fixers``
2to3 변환 중에 사용할 추가 fixer를 검색해야 할 module 목록. 자세한 것은 :doc:`python3` 를 참조.


Using ``find_packages()``
-------------------------

For simple projects, it's usually easy enough to manually add packages to
the ``packages`` argument of ``setup()``.  However, for very large projects
(Twisted, PEAK, Zope, Chandler, etc.), it can be a big burden to keep the
package list updated.  That's what ``setuptools.find_packages()`` is for.

``find_packages()`` takes a source directory and two lists of package name
patterns to exclude and include.  If omitted, the source directory defaults to
the same
directory as the setup script.  Some projects use a ``src`` or ``lib``
directory as the root of their source tree, and those projects would of course
use ``"src"`` or ``"lib"`` as the first argument to ``find_packages()``.  (And
such projects also need something like ``package_dir={'':'src'}`` in their
``setup()`` arguments, but that's just a normal distutils thing.)

Anyway, ``find_packages()`` walks the target directory, filtering by inclusion
patterns, and finds Python packages (any directory). On Python 3.2 and
earlier, packages are only recognized if they include an ``__init__.py`` file.
Finally, exclusion patterns are applied to remove matching packages.

Inclusion and exclusion patterns are package names, optionally including
wildcards.  For
example, ``find_packages(exclude=["*.tests"])`` will exclude all packages whose
last name part is ``tests``.   Or, ``find_packages(exclude=["*.tests",
"*.tests.*"])`` will also exclude any subpackages of packages named ``tests``,
but it still won't exclude a top-level ``tests`` package or the children
thereof.  In fact, if you really want no ``tests`` packages at all, you'll need
something like this::

    find_packages(exclude=["*.tests", "*.tests.*", "tests.*", "tests"])

in order to cover all the bases.  Really, the exclusion patterns are intended
to cover simpler use cases than this, like excluding a single, specified
package and its subpackages.

Regardless of the parameters, the ``find_packages()``
function returns a list of package names suitable for use as the ``packages``
argument to ``setup()``, and so is usually the easiest way to set that
argument in your setup script.  Especially since it frees you from having to
remember to modify your setup script whenever your project grows additional
top-level packages or subpackages.


Automatic Script Creation
=========================

Packaging and installing scripts can be a bit awkward with the distutils.  For
one thing, there's no easy way to have a script's filename match local
conventions on both Windows and POSIX platforms.  For another, you often have
to create a separate file just for the "main" script, when your actual "main"
is a function in a module somewhere.  And even in Python 2.4, using the ``-m``
option only works for actual ``.py`` files that aren't installed in a package.

``setuptools`` fixes all of these problems by automatically generating scripts
for you with the correct extension, and on Windows it will even create an
``.exe`` file so that users don't have to change their ``PATHEXT`` settings.
The way to use this feature is to define "entry points" in your setup script
that indicate what function the generated script should import and run.  For
example, to create two console scripts called ``foo`` and ``bar``, and a GUI
script called ``baz``, you might do something like this::

    setup(
        # other arguments here...
        entry_points={
            'console_scripts': [
                'foo = my_package.some_module:main_func',
                'bar = other_module:some_func',
            ],
            'gui_scripts': [
                'baz = my_package_gui:start_func',
            ]
        }
    )

When this project is installed on non-Windows platforms (using "setup.py
install", "setup.py develop", or by using EasyInstall), a set of ``foo``,
``bar``, and ``baz`` scripts will be installed that import ``main_func`` and
``some_func`` from the specified modules.  The functions you specify are called
with no arguments, and their return value is passed to ``sys.exit()``, so you
can return an errorlevel or message to print to stderr.

On Windows, a set of ``foo.exe``, ``bar.exe``, and ``baz.exe`` launchers are
created, alongside a set of ``foo.py``, ``bar.py``, and ``baz.pyw`` files.  The
``.exe`` wrappers find and execute the right version of Python to run the
``.py`` or ``.pyw`` file.

You may define as many "console script" and "gui script" entry points as you
like, and each one can optionally specify "extras" that it depends on, that
will be added to ``sys.path`` when the script is run.  For more information on
"extras", see the section below on `Declaring Extras`_.  For more information
on "entry points" in general, see the section below on `Dynamic Discovery of
Services and Plugins`_.


"Eggsecutable" Scripts
----------------------

Occasionally, there are situations where it's desirable to make an ``.egg``
file directly executable.  You can do this by including an entry point such
as the following::

    setup(
        # other arguments here...
        entry_points={
            'setuptools.installation': [
                'eggsecutable = my_package.some_module:main_func',
            ]
        }
    )

Any eggs built from the above setup script will include a short executable
prelude that imports and calls ``main_func()`` from ``my_package.some_module``.
The prelude can be run on Unix-like platforms (including Mac and Linux) by
invoking the egg with ``/bin/sh``, or by enabling execute permissions on the
``.egg`` file.  For the executable prelude to run, the appropriate version of
Python must be available via the ``PATH`` environment variable, under its
"long" name.  That is, if the egg is built for Python 2.3, there must be a
``python2.3`` executable present in a directory on ``PATH``.

This feature is primarily intended to support ez_setup the installation of
setuptools itself on non-Windows platforms, but may also be useful for other
projects as well.

IMPORTANT NOTE: Eggs with an "eggsecutable" header cannot be renamed, or
invoked via symlinks.  They *must* be invoked using their original filename, in
order to ensure that, once running, ``pkg_resources`` will know what project
and version is in use.  The header script will check this and exit with an
error if the ``.egg`` file has been renamed or is invoked via a symlink that
changes its base name.


Declaring Dependencies
======================

``setuptools`` supports automatically installing dependencies when a package is
installed, and including information about dependencies in Python Eggs (so that
package management tools like EasyInstall can use the information).

``setuptools`` and ``pkg_resources`` use a common syntax for specifying a
project's required dependencies.  This syntax consists of a project's PyPI
name, optionally followed by a comma-separated list of "extras" in square
brackets, optionally followed by a comma-separated list of version
specifiers.  A version specifier is one of the operators ``<``, ``>``, ``<=``,
``>=``, ``==`` or ``!=``, followed by a version identifier.  Tokens may be
separated by whitespace, but any whitespace or nonstandard characters within a
project name or version identifier must be replaced with ``-``.

Version specifiers for a given project are internally sorted into ascending
version order, and used to establish what ranges of versions are acceptable.
Adjacent redundant conditions are also consolidated (e.g. ``">1, >2"`` becomes
``">1"``, and ``"<2,<3"`` becomes ``"<3"``). ``"!="`` versions are excised from
the ranges they fall within.  A project's version is then checked for
membership in the resulting ranges. (Note that providing conflicting conditions
for the same version (e.g. "<2,>=2" or "==2,!=2") is meaningless and may
therefore produce bizarre results.)

Here are some example requirement specifiers::

    docutils >= 0.3

    # comment lines and \ continuations are allowed in requirement strings
    BazSpam ==1.1, ==1.2, ==1.3, ==1.4, ==1.5, \
        ==1.6, ==1.7  # and so are line-end comments

    PEAK[FastCGI, reST]>=0.5a4

    setuptools==0.5a7

The simplest way to include requirement specifiers is to use the
``install_requires`` argument to ``setup()``.  It takes a string or list of
strings containing requirement specifiers.  If you include more than one
requirement in a string, each requirement must begin on a new line.

This has three effects:

1. When your project is installed, either by using EasyInstall, ``setup.py
   install``, or ``setup.py develop``, all of the dependencies not already
   installed will be located (via PyPI), downloaded, built (if necessary),
   and installed.

2. Any scripts in your project will be installed with wrappers that verify
   the availability of the specified dependencies at runtime, and ensure that
   the correct versions are added to ``sys.path`` (e.g. if multiple versions
   have been installed).

3. Python Egg distributions will include a metadata file listing the
   dependencies.

Note, by the way, that if you declare your dependencies in ``setup.py``, you do
*not* need to use the ``require()`` function in your scripts or modules, as
long as you either install the project or use ``setup.py develop`` to do
development work on it.  (See `"Development Mode"`_ below for more details on
using ``setup.py develop``.)


Dependencies that aren't in PyPI
--------------------------------

If your project depends on packages that aren't registered in PyPI, you may
still be able to depend on them, as long as they are available for download
as:

- an egg, in the standard distutils ``sdist`` format,
- a single ``.py`` file, or
- a VCS repository (Subversion, Mercurial, or Git).

You just need to add some URLs to the ``dependency_links`` argument to
``setup()``.

The URLs must be either:

1. direct download URLs,
2. the URLs of web pages that contain direct download links, or
3. the repository's URL

In general, it's better to link to web pages, because it is usually less
complex to update a web page than to release a new version of your project.
You can also use a SourceForge ``showfiles.php`` link in the case where a
package you depend on is distributed via SourceForge.

If you depend on a package that's distributed as a single ``.py`` file, you
must include an ``"#egg=project-version"`` suffix to the URL, to give a project
name and version number.  (Be sure to escape any dashes in the name or version
by replacing them with underscores.)  EasyInstall will recognize this suffix
and automatically create a trivial ``setup.py`` to wrap the single ``.py`` file
as an egg.

In the case of a VCS checkout, you should also append ``#egg=project-version``
in order to identify for what package that checkout should be used. You can
append ``@REV`` to the URL's path (before the fragment) to specify a revision.
Additionally, you can also force the VCS being used by prepending the URL with
a certain prefix. Currently available are:

-  ``svn+URL`` for Subversion,
-  ``git+URL`` for Git, and
-  ``hg+URL`` for Mercurial

A more complete example would be:

    ``vcs+proto://host/path@revision#egg=project-version``

Be careful with the version. It should match the one inside the project files.
If you want to disregard the version, you have to omit it both in the
``requires`` and in the URL's fragment.

This will do a checkout (or a clone, in Git and Mercurial parlance) to a
temporary folder and run ``setup.py bdist_egg``.

The ``dependency_links`` option takes the form of a list of URL strings.  For
example, the below will cause EasyInstall to search the specified page for
eggs or source distributions, if the package's dependencies aren't already
installed::

    setup(
        ...
        dependency_links=[
            "http://peak.telecommunity.com/snapshots/"
        ],
    )


.. _Declaring Extras:


Declaring "Extras" (optional features with their own dependencies)
------------------------------------------------------------------

Sometimes a project has "recommended" dependencies, that are not required for
all uses of the project.  For example, a project might offer optional PDF
output if ReportLab is installed, and reStructuredText support if docutils is
installed.  These optional features are called "extras", and setuptools allows
you to define their requirements as well.  In this way, other projects that
require these optional features can force the additional requirements to be
installed, by naming the desired extras in their ``install_requires``.

For example, let's say that Project A offers optional PDF and reST support::

    setup(
        name="Project-A",
        ...
        extras_require={
            'PDF':  ["ReportLab>=1.2", "RXP"],
            'reST': ["docutils>=0.3"],
        }
    )

As you can see, the ``extras_require`` argument takes a dictionary mapping
names of "extra" features, to strings or lists of strings describing those
features' requirements.  These requirements will *not* be automatically
installed unless another package depends on them (directly or indirectly) by
including the desired "extras" in square brackets after the associated project
name.  (Or if the extras were listed in a requirement spec on the EasyInstall
command line.)

Extras can be used by a project's `entry points`_ to specify dynamic
dependencies.  For example, if Project A includes a "rst2pdf" script, it might
declare it like this, so that the "PDF" requirements are only resolved if the
"rst2pdf" script is run::

    setup(
        name="Project-A",
        ...
        entry_points={
            'console_scripts': [
                'rst2pdf = project_a.tools.pdfgen [PDF]',
                'rst2html = project_a.tools.htmlgen',
                # more script entry points ...
            ],
        }
    )

Projects can also use another project's extras when specifying dependencies.
For example, if project B needs "project A" with PDF support installed, it
might declare the dependency like this::

    setup(
        name="Project-B",
        install_requires=["Project-A[PDF]"],
        ...
    )

This will cause ReportLab to be installed along with project A, if project B is
installed -- even if project A was already installed.  In this way, a project
can encapsulate groups of optional "downstream dependencies" under a feature
name, so that packages that depend on it don't have to know what the downstream
dependencies are.  If a later version of Project A builds in PDF support and
no longer needs ReportLab, or if it ends up needing other dependencies besides
ReportLab in order to provide PDF support, Project B's setup information does
not need to change, but the right packages will still be installed if needed.

Note, by the way, that if a project ends up not needing any other packages to
support a feature, it should keep an empty requirements list for that feature
in its ``extras_require`` argument, so that packages depending on that feature
don't break (due to an invalid feature name).  For example, if Project A above
builds in PDF support and no longer needs ReportLab, it could change its
setup to this::

    setup(
        name="Project-A",
        ...
        extras_require={
            'PDF':  [],
            'reST': ["docutils>=0.3"],
        }
    )

so that Package B doesn't have to remove the ``[PDF]`` from its requirement
specifier.


.. _Platform Specific Dependencies:


Declaring platform specific dependencies
----------------------------------------

Sometimes a project might require a dependency to run on a specific platform.
This could to a package that back ports a module so that it can be used in
older python versions.  Or it could be a package that is required to run on a
specific operating system.  This will allow a project to work on multiple
different platforms without installing dependencies that are not required for
a platform that is installing the project.

For example, here is a project that uses the ``enum`` module and ``pywin32``::

    setup(
        name="Project",
        ...
        install_requires=[
            'enum34;python_version<"3.4"',
            'pywin32 >= 1.0;platform_system=="Windows"'
        ]
    )

Since the ``enum`` module was added in Python 3.4, it should only be installed
if the python version is earlier.  Since ``pywin32`` will only be used on
windows, it should only be installed when the operating system is Windows.
Specifying version requirements for the dependencies is supported as normal.

The environmental markers that may be used for testing platform types are
detailed in `PEP 508`_.

.. _PEP 508: https://www.python.org/dev/peps/pep-0508/

Including Data Files
====================

The distutils have traditionally allowed installation of "data files", which
are placed in a platform-specific location.  However, the most common use case
for data files distributed with a package is for use *by* the package, usually
by including the data files in the package directory.

Setuptools offers three ways to specify data files to be included in your
packages.  First, you can simply use the ``include_package_data`` keyword,
e.g.::

    from setuptools import setup, find_packages
    setup(
        ...
        include_package_data=True
    )

This tells setuptools to install any data files it finds in your packages.
The data files must be specified via the distutils' ``MANIFEST.in`` file.
(They can also be tracked by a revision control system, using an appropriate
plugin.  See the section below on `Adding Support for Revision Control
Systems`_ for information on how to write such plugins.)

If you want finer-grained control over what files are included (for example,
if you have documentation files in your package directories and want to exclude
them from installation), then you can also use the ``package_data`` keyword,
e.g.::

    from setuptools import setup, find_packages
    setup(
        ...
        package_data={
            # If any package contains *.txt or *.rst files, include them:
            '': ['*.txt', '*.rst'],
            # And include any *.msg files found in the 'hello' package, too:
            'hello': ['*.msg'],
        }
    )

The ``package_data`` argument is a dictionary that maps from package names to
lists of glob patterns.  The globs may include subdirectory names, if the data
files are contained in a subdirectory of the package.  For example, if the
package tree looks like this::

    setup.py
    src/
        mypkg/
            __init__.py
            mypkg.txt
            data/
                somefile.dat
                otherdata.dat

The setuptools setup file might look like this::

    from setuptools import setup, find_packages
    setup(
        ...
        packages=find_packages('src'),  # include all packages under src
        package_dir={'':'src'},   # tell distutils packages are under src

        package_data={
            # If any package contains *.txt files, include them:
            '': ['*.txt'],
            # And include any *.dat files found in the 'data' subdirectory
            # of the 'mypkg' package, also:
            'mypkg': ['data/*.dat'],
        }
    )

Notice that if you list patterns in ``package_data`` under the empty string,
these patterns are used to find files in every package, even ones that also
have their own patterns listed.  Thus, in the above example, the ``mypkg.txt``
file gets included even though it's not listed in the patterns for ``mypkg``.

Also notice that if you use paths, you *must* use a forward slash (``/``) as
the path separator, even if you are on Windows.  Setuptools automatically
converts slashes to appropriate platform-specific separators at build time.

(Note: although the ``package_data`` argument was previously only available in
``setuptools``, it was also added to the Python ``distutils`` package as of
Python 2.4; there is `some documentation for the feature`__ available on the
python.org website.  If using the setuptools-specific ``include_package_data``
argument, files specified by ``package_data`` will *not* be automatically
added to the manifest unless they are listed in the MANIFEST.in file.)

__ http://docs.python.org/dist/node11.html

Sometimes, the ``include_package_data`` or ``package_data`` options alone
aren't sufficient to precisely define what files you want included.  For
example, you may want to include package README files in your revision control
system and source distributions, but exclude them from being installed.  So,
setuptools offers an ``exclude_package_data`` option as well, that allows you
to do things like this::

    from setuptools import setup, find_packages
    setup(
        ...
        packages=find_packages('src'),  # include all packages under src
        package_dir={'':'src'},   # tell distutils packages are under src

        include_package_data=True,    # include everything in source control

        # ...but exclude README.txt from all packages
        exclude_package_data={'': ['README.txt']},
    )

The ``exclude_package_data`` option is a dictionary mapping package names to
lists of wildcard patterns, just like the ``package_data`` option.  And, just
as with that option, a key of ``''`` will apply the given pattern(s) to all
packages.  However, any files that match these patterns will be *excluded*
from installation, even if they were listed in ``package_data`` or were
included as a result of using ``include_package_data``.

In summary, the three options allow you to:

``include_package_data``
    Accept all data files and directories matched by ``MANIFEST.in``.

``package_data``
    Specify additional patterns to match files and directories that may or may
    not be matched by ``MANIFEST.in`` or found in source control.

``exclude_package_data``
    Specify patterns for data files and directories that should *not* be
    included when a package is installed, even if they would otherwise have
    been included due to the use of the preceding options.

NOTE: Due to the way the distutils build process works, a data file that you
include in your project and then stop including may be "orphaned" in your
project's build directories, requiring you to run ``setup.py clean --all`` to
fully remove them.  This may also be important for your users and contributors
if they track intermediate revisions of your project using Subversion; be sure
to let them know when you make changes that remove files from inclusion so they
can run ``setup.py clean --all``.


Accessing Data Files at Runtime
-------------------------------

Typically, existing programs manipulate a package's ``__file__`` attribute in
order to find the location of data files.  However, this manipulation isn't
compatible with PEP 302-based import hooks, including importing from zip files
and Python Eggs.  It is strongly recommended that, if you are using data files,
you should use the :ref:`ResourceManager API` of ``pkg_resources`` to access
them.  The ``pkg_resources`` module is distributed as part of setuptools, so if
you're using setuptools to distribute your package, there is no reason not to
use its resource management API.  See also `Accessing Package Resources`_ for
a quick example of converting code that uses ``__file__`` to use
``pkg_resources`` instead.

.. _Accessing Package Resources: http://peak.telecommunity.com/DevCenter/PythonEggs#accessing-package-resources


Non-Package Data Files
----------------------

The ``distutils`` normally install general "data files" to a platform-specific
location (e.g. ``/usr/share``).  This feature intended to be used for things
like documentation, example configuration files, and the like.  ``setuptools``
does not install these data files in a separate location, however.  They are
bundled inside the egg file or directory, alongside the Python modules and
packages.  The data files can also be accessed using the :ref:`ResourceManager
API`, by specifying a ``Requirement`` instead of a package name::

    from pkg_resources import Requirement, resource_filename
    filename = resource_filename(Requirement.parse("MyProject"),"sample.conf")

The above code will obtain the filename of the "sample.conf" file in the data
root of the "MyProject" distribution.

Note, by the way, that this encapsulation of data files means that you can't
actually install data files to some arbitrary location on a user's machine;
this is a feature, not a bug.  You can always include a script in your
distribution that extracts and copies your the documentation or data files to
a user-specified location, at their discretion.  If you put related data files
in a single directory, you can use ``resource_filename()`` with the directory
name to get a filesystem directory that then can be copied with the ``shutil``
module.  (Even if your package is installed as a zipfile, calling
``resource_filename()`` on a directory will return an actual filesystem
directory, whose contents will be that entire subtree of your distribution.)

(Of course, if you're writing a new package, you can just as easily place your
data files or directories inside one of your packages, rather than using the
distutils' approach.  However, if you're updating an existing application, it
may be simpler not to change the way it currently specifies these data files.)


Automatic Resource Extraction
-----------------------------

If you are using tools that expect your resources to be "real" files, or your
project includes non-extension native libraries or other files that your C
extensions expect to be able to access, you may need to list those files in
the ``eager_resources`` argument to ``setup()``, so that the files will be
extracted together, whenever a C extension in the project is imported.

This is especially important if your project includes shared libraries *other*
than distutils-built C extensions, and those shared libraries use file
extensions other than ``.dll``, ``.so``, or ``.dylib``, which are the
extensions that setuptools 0.6a8 and higher automatically detects as shared
libraries and adds to the ``native_libs.txt`` file for you.  Any shared
libraries whose names do not end with one of those extensions should be listed
as ``eager_resources``, because they need to be present in the filesystem when
he C extensions that link to them are used.

The ``pkg_resources`` runtime for compressed packages will automatically
extract *all* C extensions and ``eager_resources`` at the same time, whenever
*any* C extension or eager resource is requested via the ``resource_filename()``
API.  (C extensions are imported using ``resource_filename()`` internally.)
This ensures that C extensions will see all of the "real" files that they
expect to see.

Note also that you can list directory resource names in ``eager_resources`` as
well, in which case the directory's contents (including subdirectories) will be
extracted whenever any C extension or eager resource is requested.

Please note that if you're not sure whether you need to use this argument, you
don't!  It's really intended to support projects with lots of non-Python
dependencies and as a last resort for crufty projects that can't otherwise
handle being compressed.  If your package is pure Python, Python plus data
files, or Python plus C, you really don't need this.  You've got to be using
either C or an external program that needs "real" files in your project before
there's any possibility of ``eager_resources`` being relevant to your project.


Extensible Applications and Frameworks
======================================


.. _Entry Points:

Dynamic Discovery of Services and Plugins
-----------------------------------------

``setuptools`` supports creating libraries that "plug in" to extensible
applications and frameworks, by letting you register "entry points" in your
project that can be imported by the application or framework.

For example, suppose that a blogging tool wants to support plugins
that provide translation for various file types to the blog's output format.
The framework might define an "entry point group" called ``blogtool.parsers``,
and then allow plugins to register entry points for the file extensions they
support.

This would allow people to create distributions that contain one or more
parsers for different file types, and then the blogging tool would be able to
find the parsers at runtime by looking up an entry point for the file
extension (or mime type, or however it wants to).

Note that if the blogging tool includes parsers for certain file formats, it
can register these as entry points in its own setup script, which means it
doesn't have to special-case its built-in formats.  They can just be treated
the same as any other plugin's entry points would be.

If you're creating a project that plugs in to an existing application or
framework, you'll need to know what entry points or entry point groups are
defined by that application or framework.  Then, you can register entry points
in your setup script.  Here are a few examples of ways you might register an
``.rst`` file parser entry point in the ``blogtool.parsers`` entry point group,
for our hypothetical blogging tool::

    setup(
        # ...
        entry_points={'blogtool.parsers': '.rst = some_module:SomeClass'}
    )

    setup(
        # ...
        entry_points={'blogtool.parsers': ['.rst = some_module:a_func']}
    )

    setup(
        # ...
        entry_points="""
            [blogtool.parsers]
            .rst = some.nested.module:SomeClass.some_classmethod [reST]
        """,
        extras_require=dict(reST="Docutils>=0.3.5")
    )

The ``entry_points`` argument to ``setup()`` accepts either a string with
``.ini``-style sections, or a dictionary mapping entry point group names to
either strings or lists of strings containing entry point specifiers.  An
entry point specifier consists of a name and value, separated by an ``=``
sign.  The value consists of a dotted module name, optionally followed by a
``:`` and a dotted identifier naming an object within the module.  It can
also include a bracketed list of "extras" that are required for the entry
point to be used.  When the invoking application or framework requests loading
of an entry point, any requirements implied by the associated extras will be
passed to ``pkg_resources.require()``, so that an appropriate error message
can be displayed if the needed package(s) are missing.  (Of course, the
invoking app or framework can ignore such errors if it wants to make an entry
point optional if a requirement isn't installed.)


Defining Additional Metadata
----------------------------

Some extensible applications and frameworks may need to define their own kinds
of metadata to include in eggs, which they can then access using the
``pkg_resources`` metadata APIs.  Ordinarily, this is done by having plugin
developers include additional files in their ``ProjectName.egg-info``
directory.  However, since it can be tedious to create such files by hand, you
may want to create a distutils extension that will create the necessary files
from arguments to ``setup()``, in much the same way that ``setuptools`` does
for many of the ``setup()`` arguments it adds.  See the section below on
`Creating distutils Extensions`_ for more details, especially the subsection on
`Adding new EGG-INFO Files`_.


"Development Mode"
==================

Under normal circumstances, the ``distutils`` assume that you are going to
build a distribution of your project, not use it in its "raw" or "unbuilt"
form.  If you were to use the ``distutils`` that way, you would have to rebuild
and reinstall your project every time you made a change to it during
development.

Another problem that sometimes comes up with the ``distutils`` is that you may
need to do development on two related projects at the same time.  You may need
to put both projects' packages in the same directory to run them, but need to
keep them separate for revision control purposes.  How can you do this?

Setuptools allows you to deploy your projects for use in a common directory or
staging area, but without copying any files.  Thus, you can edit each project's
code in its checkout directory, and only need to run build commands when you
change a project's C extensions or similarly compiled files.  You can even
deploy a project into another project's checkout directory, if that's your
preferred way of working (as opposed to using a common independent staging area
or the site-packages directory).

To do this, use the ``setup.py develop`` command.  It works very similarly to
``setup.py install`` or the EasyInstall tool, except that it doesn't actually
install anything.  Instead, it creates a special ``.egg-link`` file in the
deployment directory, that links to your project's source code.  And, if your
deployment directory is Python's ``site-packages`` directory, it will also
update the ``easy-install.pth`` file to include your project's source code,
thereby making it available on ``sys.path`` for all programs using that Python
installation.

If you have enabled the ``use_2to3`` flag, then of course the ``.egg-link``
will not link directly to your source code when run under Python 3, since
that source code would be made for Python 2 and not work under Python 3.
Instead the ``setup.py develop`` will build Python 3 code under the ``build``
directory, and link there. This means that after doing code changes you will
have to run ``setup.py build`` before these changes are picked up by your
Python 3 installation.

In addition, the ``develop`` command creates wrapper scripts in the target
script directory that will run your in-development scripts after ensuring that
all your ``install_requires`` packages are available on ``sys.path``.

You can deploy the same project to multiple staging areas, e.g. if you have
multiple projects on the same machine that are sharing the same project you're
doing development work.

When you're done with a given development task, you can remove the project
source from a staging area using ``setup.py develop --uninstall``, specifying
the desired staging area if it's not the default.

There are several options to control the precise behavior of the ``develop``
command; see the section on the `develop`_ command below for more details.

Note that you can also apply setuptools commands to non-setuptools projects,
using commands like this::

   python -c "import setuptools; execfile('setup.py')" develop

That is, you can simply list the normal setup commands and options following
the quoted part.


Distributing a ``setuptools``-based project
===========================================

Using ``setuptools``...  Without bundling it!
---------------------------------------------

.. warning:: **ez_setup** is deprecated in favor of PIP with **PEP-518** support.

Your users might not have ``setuptools`` installed on their machines, or even
if they do, it might not be the right version.  Fixing this is easy; just
download `ez_setup.py`_, and put it in the same directory as your ``setup.py``
script.  (Be sure to add it to your revision control system, too.)  Then add
these two lines to the very top of your setup script, before the script imports
anything from setuptools:

.. code-block:: python

    import ez_setup
    ez_setup.use_setuptools()

That's it.  The ``ez_setup`` module will automatically download a matching
version of ``setuptools`` from PyPI, if it isn't present on the target system.
Whenever you install an updated version of setuptools, you should also update
your projects' ``ez_setup.py`` files, so that a matching version gets installed
on the target machine(s).

By the way, setuptools supports the new PyPI "upload" command, so you can use
``setup.py sdist upload`` or ``setup.py bdist_egg upload`` to upload your
source or egg distributions respectively.  Your project's current version must
be registered with PyPI first, of course; you can use ``setup.py register`` to
do that.  Or you can do it all in one step, e.g. ``setup.py register sdist
bdist_egg upload`` will register the package, build source and egg
distributions, and then upload them both to PyPI, where they'll be easily
found by other projects that depend on them.

(By the way, if you need to distribute a specific version of ``setuptools``,
you can specify the exact version and base download URL as parameters to the
``use_setuptools()`` function.  See the function's docstring for details.)


What Your Users Should Know
---------------------------

In general, a setuptools-based project looks just like any distutils-based
project -- as long as your users have an internet connection and are installing
to ``site-packages``, that is.  But for some users, these conditions don't
apply, and they may become frustrated if this is their first encounter with
a setuptools-based project.  To keep these users happy, you should review the
following topics in your project's installation instructions, if they are
relevant to your project and your target audience isn't already familiar with
setuptools and ``easy_install``.

Network Access
    If your project is using ``ez_setup``, you should inform users of the
    need to either have network access, or to preinstall the correct version of
    setuptools using the `EasyInstall installation instructions`_.  Those
    instructions also have tips for dealing with firewalls as well as how to
    manually download and install setuptools.

Custom Installation Locations
    You should inform your users that if they are installing your project to
    somewhere other than the main ``site-packages`` directory, they should
    first install setuptools using the instructions for `Custom Installation
    Locations`_, before installing your project.

Your Project's Dependencies
    If your project depends on other projects that may need to be downloaded
    from PyPI or elsewhere, you should list them in your installation
    instructions, or tell users how to find out what they are.  While most
    users will not need this information, any users who don't have unrestricted
    internet access may have to find, download, and install the other projects
    manually.  (Note, however, that they must still install those projects
    using ``easy_install``, or your project will not know they are installed,
    and your setup script will try to download them again.)

    If you want to be especially friendly to users with limited network access,
    you may wish to build eggs for your project and its dependencies, making
    them all available for download from your site, or at least create a page
    with links to all of the needed eggs.  In this way, users with limited
    network access can manually download all the eggs to a single directory,
    then use the ``-f`` option of ``easy_install`` to specify the directory
    to find eggs in.  Users who have full network access can just use ``-f``
    with the URL of your download page, and ``easy_install`` will find all the
    needed eggs using your links directly.  This is also useful when your
    target audience isn't able to compile packages (e.g. most Windows users)
    and your package or some of its dependencies include C code.

Revision Control System Users and Co-Developers
    Users and co-developers who are tracking your in-development code using
    a revision control system should probably read this manual's sections
    regarding such development.  Alternately, you may wish to create a
    quick-reference guide containing the tips from this manual that apply to
    your particular situation.  For example, if you recommend that people use
    ``setup.py develop`` when tracking your in-development code, you should let
    them know that this needs to be run after every update or commit.

    Similarly, if you remove modules or data files from your project, you
    should remind them to run ``setup.py clean --all`` and delete any obsolete
    ``.pyc`` or ``.pyo``.  (This tip applies to the distutils in general, not
    just setuptools, but not everybody knows about them; be kind to your users
    by spelling out your project's best practices rather than leaving them
    guessing.)

Creating System Packages
    Some users want to manage all Python packages using a single package
    manager, and sometimes that package manager isn't ``easy_install``!
    Setuptools currently supports ``bdist_rpm``, ``bdist_wininst``, and
    ``bdist_dumb`` formats for system packaging.  If a user has a locally-
    installed "bdist" packaging tool that internally uses the distutils
    ``install`` command, it should be able to work with ``setuptools``.  Some
    examples of "bdist" formats that this should work with include the
    ``bdist_nsi`` and ``bdist_msi`` formats for Windows.

    However, packaging tools that build binary distributions by running
    ``setup.py install`` on the command line or as a subprocess will require
    modification to work with setuptools.  They should use the
    ``--single-version-externally-managed`` option to the ``install`` command,
    combined with the standard ``--root`` or ``--record`` options.
    See the `install command`_ documentation below for more details.  The
    ``bdist_deb`` command is an example of a command that currently requires
    this kind of patching to work with setuptools.

    If you or your users have a problem building a usable system package for
    your project, please report the problem via the mailing list so that
    either the "bdist" tool in question or setuptools can be modified to
    resolve the issue.


Setting the ``zip_safe`` flag
-----------------------------

For some use cases (such as bundling as part of a larger application), Python
packages may be run directly from a zip file.
Not all packages, however, are capable of running in compressed form, because
they may expect to be able to access either source code or data files as
normal operating system files.  So, ``setuptools`` can install your project
as a zipfile or a directory, and its default choice is determined by the
project's ``zip_safe`` flag.

You can pass a True or False value for the ``zip_safe`` argument to the
``setup()`` function, or you can omit it.  If you omit it, the ``bdist_egg``
command will analyze your project's contents to see if it can detect any
conditions that would prevent it from working in a zipfile.  It will output
notices to the console about any such conditions that it finds.

Currently, this analysis is extremely conservative: it will consider the
project unsafe if it contains any C extensions or datafiles whatsoever.  This
does *not* mean that the project can't or won't work as a zipfile!  It just
means that the ``bdist_egg`` authors aren't yet comfortable asserting that
the project *will* work.  If the project contains no C or data files, and does
no ``__file__`` or ``__path__`` introspection or source code manipulation, then
there is an extremely solid chance the project will work when installed as a
zipfile.  (And if the project uses ``pkg_resources`` for all its data file
access, then C extensions and other data files shouldn't be a problem at all.
See the `Accessing Data Files at Runtime`_ section above for more information.)

However, if ``bdist_egg`` can't be *sure* that your package will work, but
you've checked over all the warnings it issued, and you are either satisfied it
*will* work (or if you want to try it for yourself), then you should set
``zip_safe`` to ``True`` in your ``setup()`` call.  If it turns out that it
doesn't work, you can always change it to ``False``, which will force
``setuptools`` to install your project as a directory rather than as a zipfile.

Of course, the end-user can still override either decision, if they are using
EasyInstall to install your package.  And, if you want to override for testing
purposes, you can just run ``setup.py easy_install --zip-ok .`` or ``setup.py
easy_install --always-unzip .`` in your project directory. to install the
package as a zipfile or directory, respectively.

In the future, as we gain more experience with different packages and become
more satisfied with the robustness of the ``pkg_resources`` runtime, the
"zip safety" analysis may become less conservative.  However, we strongly
recommend that you determine for yourself whether your project functions
correctly when installed as a zipfile, correct any problems if you can, and
then make an explicit declaration of ``True`` or ``False`` for the ``zip_safe``
flag, so that it will not be necessary for ``bdist_egg`` or ``EasyInstall`` to
try to guess whether your project can work as a zipfile.


Namespace Packages
------------------

Sometimes, a large package is more useful if distributed as a collection of
smaller eggs.  However, Python does not normally allow the contents of a
package to be retrieved from more than one location.  "Namespace packages"
are a solution for this problem.  When you declare a package to be a namespace
package, it means that the package has no meaningful contents in its
``__init__.py``, and that it is merely a container for modules and subpackages.

The ``pkg_resources`` runtime will then automatically ensure that the contents
of namespace packages that are spread over multiple eggs or directories are
combined into a single "virtual" package.

The ``namespace_packages`` argument to ``setup()`` lets you declare your
project's namespace packages, so that they will be included in your project's
metadata.  The argument should list the namespace packages that the egg
participates in.  For example, the ZopeInterface project might do this::

    setup(
        # ...
        namespace_packages=['zope']
    )

because it contains a ``zope.interface`` package that lives in the ``zope``
namespace package.  Similarly, a project for a standalone ``zope.publisher``
would also declare the ``zope`` namespace package.  When these projects are
installed and used, Python will see them both as part of a "virtual" ``zope``
package, even though they will be installed in different locations.

Namespace packages don't have to be top-level packages.  For example, Zope 3's
``zope.app`` package is a namespace package, and in the future PEAK's
``peak.util`` package will be too.

Note, by the way, that your project's source tree must include the namespace
packages' ``__init__.py`` files (and the ``__init__.py`` of any parent
packages), in a normal Python package layout.  These ``__init__.py`` files
*must* contain the line::

    __import__('pkg_resources').declare_namespace(__name__)

This code ensures that the namespace package machinery is operating and that
the current package is registered as a namespace package.

You must NOT include any other code and data in a namespace package's
``__init__.py``.  Even though it may appear to work during development, or when
projects are installed as ``.egg`` files, it will not work when the projects
are installed using "system" packaging tools -- in such cases the
``__init__.py`` files will not be installed, let alone executed.

You must include the ``declare_namespace()``  line in the ``__init__.py`` of
*every* project that has contents for the namespace package in question, in
order to ensure that the namespace will be declared regardless of which
project's copy of ``__init__.py`` is loaded first.  If the first loaded
``__init__.py`` doesn't declare it, it will never *be* declared, because no
other copies will ever be loaded!


TRANSITIONAL NOTE
~~~~~~~~~~~~~~~~~

Setuptools automatically calls ``declare_namespace()`` for you at runtime,
but future versions may *not*.  This is because the automatic declaration
feature has some negative side effects, such as needing to import all namespace
packages during the initialization of the ``pkg_resources`` runtime, and also
the need for ``pkg_resources`` to be explicitly imported before any namespace
packages work at all.  In some future releases, you'll be responsible
for including your own declaration lines, and the automatic declaration feature
will be dropped to get rid of the negative side effects.

During the remainder of the current development cycle, therefore, setuptools
will warn you about missing ``declare_namespace()`` calls in your
``__init__.py`` files, and you should correct these as soon as possible
before the compatibility support is removed.
Namespace packages without declaration lines will not work
correctly once a user has upgraded to a later version, so it's important that
you make this change now in order to avoid having your code break in the field.
Our apologies for the inconvenience, and thank you for your patience.



Tagging and "Daily Build" or "Snapshot" Releases
------------------------------------------------

When a set of related projects are under development, it may be important to
track finer-grained version increments than you would normally use for e.g.
"stable" releases.  While stable releases might be measured in dotted numbers
with alpha/beta/etc. status codes, development versions of a project often
need to be tracked by revision or build number or even build date.  This is
especially true when projects in development need to refer to one another, and
therefore may literally need an up-to-the-minute version of something!

To support these scenarios, ``setuptools`` allows you to "tag" your source and
egg distributions by adding one or more of the following to the project's
"official" version identifier:

* A manually-specified pre-release tag, such as "build" or "dev", or a
  manually-specified post-release tag, such as a build or revision number
  (``--tag-build=STRING, -bSTRING``)

* An 8-character representation of the build date (``--tag-date, -d``), as
  a postrelease tag

You can add these tags by adding ``egg_info`` and the desired options to
the command line ahead of the ``sdist`` or ``bdist`` commands that you want
to generate a daily build or snapshot for.  See the section below on the
`egg_info`_ command for more details.

(Also, before you release your project, be sure to see the section above on
`Specifying Your Project's Version`_ for more information about how pre- and
post-release tags affect how setuptools and EasyInstall interpret version
numbers.  This is important in order to make sure that dependency processing
tools will know which versions of your project are newer than others.)

Finally, if you are creating builds frequently, and either building them in a
downloadable location or are copying them to a distribution server, you should
probably also check out the `rotate`_ command, which lets you automatically
delete all but the N most-recently-modified distributions matching a glob
pattern.  So, you can use a command line like::

    setup.py egg_info -rbDEV bdist_egg rotate -m.egg -k3

to build an egg whose version info includes 'DEV-rNNNN' (where NNNN is the
most recent Subversion revision that affected the source tree), and then
delete any egg files from the distribution directory except for the three
that were built most recently.

If you have to manage automated builds for multiple packages, each with
different tagging and rotation policies, you may also want to check out the
`alias`_ command, which would let each package define an alias like ``daily``
that would perform the necessary tag, build, and rotate commands.  Then, a
simpler script or cron job could just run ``setup.py daily`` in each project
directory.  (And, you could also define sitewide or per-user default versions
of the ``daily`` alias, so that projects that didn't define their own would
use the appropriate defaults.)


Generating Source Distributions
-------------------------------

``setuptools`` enhances the distutils' default algorithm for source file
selection with pluggable endpoints for looking up files to include. If you are
using a revision control system, and your source distributions only need to
include files that you're tracking in revision control, use a corresponding
plugin instead of writing a ``MANIFEST.in`` file. See the section below on
`Adding Support for Revision Control Systems`_ for information on plugins.

If you need to include automatically generated files, or files that are kept in
an unsupported revision control system, you'll need to create a ``MANIFEST.in``
file to specify any files that the default file location algorithm doesn't
catch.  See the distutils documentation for more information on the format of
the ``MANIFEST.in`` file.

But, be sure to ignore any part of the distutils documentation that deals with
``MANIFEST`` or how it's generated from ``MANIFEST.in``; setuptools shields you
from these issues and doesn't work the same way in any case.  Unlike the
distutils, setuptools regenerates the source distribution manifest file
every time you build a source distribution, and it builds it inside the
project's ``.egg-info`` directory, out of the way of your main project
directory.  You therefore need not worry about whether it is up-to-date or not.

Indeed, because setuptools' approach to determining the contents of a source
distribution is so much simpler, its ``sdist`` command omits nearly all of
the options that the distutils' more complex ``sdist`` process requires.  For
all practical purposes, you'll probably use only the ``--formats`` option, if
you use any option at all.


Making your package available for EasyInstall
---------------------------------------------

If you use the ``register`` command (``setup.py register``) to register your
package with PyPI, that's most of the battle right there.  (See the
`docs for the register command`_ for more details.)

.. _docs for the register command: http://docs.python.org/dist/package-index.html

If you also use the `upload`_ command to upload actual distributions of your
package, that's even better, because EasyInstall will be able to find and
download them directly from your project's PyPI page.

However, there may be reasons why you don't want to upload distributions to
PyPI, and just want your existing distributions (or perhaps a Subversion
checkout) to be used instead.

So here's what you need to do before running the ``register`` command.  There
are three ``setup()`` arguments that affect EasyInstall:

``url`` and ``download_url``
   These become links on your project's PyPI page.  EasyInstall will examine
   them to see if they link to a package ("primary links"), or whether they are
   HTML pages.  If they're HTML pages, EasyInstall scans all HREF's on the
   page for primary links

``long_description``
   EasyInstall will check any URLs contained in this argument to see if they
   are primary links.

A URL is considered a "primary link" if it is a link to a .tar.gz, .tgz, .zip,
.egg, .egg.zip, .tar.bz2, or .exe file, or if it has an ``#egg=project`` or
``#egg=project-version`` fragment identifier attached to it.  EasyInstall
attempts to determine a project name and optional version number from the text
of a primary link *without* downloading it.  When it has found all the primary
links, EasyInstall will select the best match based on requested version,
platform compatibility, and other criteria.

So, if your ``url`` or ``download_url`` point either directly to a downloadable
source distribution, or to HTML page(s) that have direct links to such, then
EasyInstall will be able to locate downloads automatically.  If you want to
make Subversion checkouts available, then you should create links with either
``#egg=project`` or ``#egg=project-version`` added to the URL.  You should
replace ``project`` and ``version`` with the values they would have in an egg
filename.  (Be sure to actually generate an egg and then use the initial part
of the filename, rather than trying to guess what the escaped form of the
project name and version number will be.)

Note that Subversion checkout links are of lower precedence than other kinds
of distributions, so EasyInstall will not select a Subversion checkout for
downloading unless it has a version included in the ``#egg=`` suffix, and
it's a higher version than EasyInstall has seen in any other links for your
project.

As a result, it's a common practice to use mark checkout URLs with a version of
"dev" (i.e., ``#egg=projectname-dev``), so that users can do something like
this::

    easy_install --editable projectname==dev

in order to check out the in-development version of ``projectname``.


Making "Official" (Non-Snapshot) Releases
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When you make an official release, creating source or binary distributions,
you will need to override the tag settings from ``setup.cfg``, so that you
don't end up registering versions like ``foobar-0.7a1.dev-r34832``.  This is
easy to do if you are developing on the trunk and using tags or branches for
your releases - just make the change to ``setup.cfg`` after branching or
tagging the release, so the trunk will still produce development snapshots.

Alternately, if you are not branching for releases, you can override the
default version options on the command line, using something like::

    python setup.py egg_info -Db "" sdist bdist_egg register upload

The first part of this command (``egg_info -Db ""``) will override the
configured tag information, before creating source and binary eggs, registering
the project with PyPI, and uploading the files.  Thus, these commands will use
the plain version from your ``setup.py``, without adding the build designation
string.

Of course, if you will be doing this a lot, you may wish to create a personal
alias for this operation, e.g.::

    python setup.py alias -u release egg_info -Db ""

You can then use it like this::

    python setup.py release sdist bdist_egg register upload

Or of course you can create more elaborate aliases that do all of the above.
See the sections below on the `egg_info`_ and `alias`_ commands for more ideas.



Distributing Extensions compiled with Pyrex
-------------------------------------------

``setuptools`` includes transparent support for building Pyrex extensions, as
long as you define your extensions using ``setuptools.Extension``, *not*
``distutils.Extension``.  You must also not import anything from Pyrex in
your setup script.

If you follow these rules, you can safely list ``.pyx`` files as the source
of your ``Extension`` objects in the setup script.  ``setuptools`` will detect
at build time whether Pyrex is installed or not.  If it is, then ``setuptools``
will use it.  If not, then ``setuptools`` will silently change the
``Extension`` objects to refer to the ``.c`` counterparts of the ``.pyx``
files, so that the normal distutils C compilation process will occur.

Of course, for this to work, your source distributions must include the C
code generated by Pyrex, as well as your original ``.pyx`` files.  This means
that you will probably want to include current ``.c`` files in your revision
control system, rebuilding them whenever you check changes in for the ``.pyx``
source files.  This will ensure that people tracking your project in a revision
control system will be able to build it even if they don't have Pyrex
installed, and that your source releases will be similarly usable with or
without Pyrex.


-----------------
Command Reference
-----------------

.. _alias:

``alias`` - Define shortcuts for commonly used commands
=======================================================

Sometimes, you need to use the same commands over and over, but you can't
necessarily set them as defaults.  For example, if you produce both development
snapshot releases and "stable" releases of a project, you may want to put
the distributions in different places, or use different ``egg_info`` tagging
options, etc.  In these cases, it doesn't make sense to set the options in
a distutils configuration file, because the values of the options changed based
on what you're trying to do.

Setuptools therefore allows you to define "aliases" - shortcut names for
an arbitrary string of commands and options, using ``setup.py alias aliasname
expansion``, where aliasname is the name of the new alias, and the remainder of
the command line supplies its expansion.  For example, this command defines
a sitewide alias called "daily", that sets various ``egg_info`` tagging
options::

    setup.py alias --global-config daily egg_info --tag-build=development

Once the alias is defined, it can then be used with other setup commands,
e.g.::

    setup.py daily bdist_egg        # generate a daily-build .egg file
    setup.py daily sdist            # generate a daily-build source distro
    setup.py daily sdist bdist_egg  # generate both

The above commands are interpreted as if the word ``daily`` were replaced with
``egg_info --tag-build=development``.

Note that setuptools will expand each alias *at most once* in a given command
line.  This serves two purposes.  First, if you accidentally create an alias
loop, it will have no effect; you'll instead get an error message about an
unknown command.  Second, it allows you to define an alias for a command, that
uses that command.  For example, this (project-local) alias::

    setup.py alias bdist_egg bdist_egg rotate -k1 -m.egg

redefines the ``bdist_egg`` command so that it always runs the ``rotate``
command afterwards to delete all but the newest egg file.  It doesn't loop
indefinitely on ``bdist_egg`` because the alias is only expanded once when
used.

You can remove a defined alias with the ``--remove`` (or ``-r``) option, e.g.::

    setup.py alias --global-config --remove daily

would delete the "daily" alias we defined above.

Aliases can be defined on a project-specific, per-user, or sitewide basis.  The
default is to define or remove a project-specific alias, but you can use any of
the `configuration file options`_ (listed under the `saveopts`_ command, below)
to determine which distutils configuration file an aliases will be added to
(or removed from).

Note that if you omit the "expansion" argument to the ``alias`` command,
you'll get output showing that alias' current definition (and what
configuration file it's defined in).  If you omit the alias name as well,
you'll get a listing of all current aliases along with their configuration
file locations.


``bdist_egg`` - Create a Python Egg for the project
===================================================

This command generates a Python Egg (``.egg`` file) for the project.  Python
Eggs are the preferred binary distribution format for EasyInstall, because they
are cross-platform (for "pure" packages), directly importable, and contain
project metadata including scripts and information about the project's
dependencies.  They can be simply downloaded and added to ``sys.path``
directly, or they can be placed in a directory on ``sys.path`` and then
automatically discovered by the egg runtime system.

This command runs the `egg_info`_ command (if it hasn't already run) to update
the project's metadata (``.egg-info``) directory.  If you have added any extra
metadata files to the ``.egg-info`` directory, those files will be included in
the new egg file's metadata directory, for use by the egg runtime system or by
any applications or frameworks that use that metadata.

You won't usually need to specify any special options for this command; just
use ``bdist_egg`` and you're done.  But there are a few options that may
be occasionally useful:

``--dist-dir=DIR, -d DIR``
    Set the directory where the ``.egg`` file will be placed.  If you don't
    supply this, then the ``--dist-dir`` setting of the ``bdist`` command
    will be used, which is usually a directory named ``dist`` in the project
    directory.

``--plat-name=PLATFORM, -p PLATFORM``
    Set the platform name string that will be embedded in the egg's filename
    (assuming the egg contains C extensions).  This can be used to override
    the distutils default platform name with something more meaningful.  Keep
    in mind, however, that the egg runtime system expects to see eggs with
    distutils platform names, so it may ignore or reject eggs with non-standard
    platform names.  Similarly, the EasyInstall program may ignore them when
    searching web pages for download links.  However, if you are
    cross-compiling or doing some other unusual things, you might find a use
    for this option.

``--exclude-source-files``
    Don't include any modules' ``.py`` files in the egg, just compiled Python,
    C, and data files.  (Note that this doesn't affect any ``.py`` files in the
    EGG-INFO directory or its subdirectories, since for example there may be
    scripts with a ``.py`` extension which must still be retained.)  We don't
    recommend that you use this option except for packages that are being
    bundled for proprietary end-user applications, or for "embedded" scenarios
    where space is at an absolute premium.  On the other hand, if your package
    is going to be installed and used in compressed form, you might as well
    exclude the source because Python's ``traceback`` module doesn't currently
    understand how to display zipped source code anyway, or how to deal with
    files that are in a different place from where their code was compiled.

There are also some options you will probably never need, but which are there
because they were copied from similar ``bdist`` commands used as an example for
creating this one.  They may be useful for testing and debugging, however,
which is why we kept them:

``--keep-temp, -k``
    Keep the contents of the ``--bdist-dir`` tree around after creating the
    ``.egg`` file.

``--bdist-dir=DIR, -b DIR``
    Set the temporary directory for creating the distribution.  The entire
    contents of this directory are zipped to create the ``.egg`` file, after
    running various installation commands to copy the package's modules, data,
    and extensions here.

``--skip-build``
    Skip doing any "build" commands; just go straight to the
    install-and-compress phases.


.. _develop:

``develop`` - Deploy the project source in "Development Mode"
=============================================================

This command allows you to deploy your project's source for use in one or more
"staging areas" where it will be available for importing.  This deployment is
done in such a way that changes to the project source are immediately available
in the staging area(s), without needing to run a build or install step after
each change.

The ``develop`` command works by creating an ``.egg-link`` file (named for the
project) in the given staging area.  If the staging area is Python's
``site-packages`` directory, it also updates an ``easy-install.pth`` file so
that the project is on ``sys.path`` by default for all programs run using that
Python installation.

The ``develop`` command also installs wrapper scripts in the staging area (or
a separate directory, as specified) that will ensure the project's dependencies
are available on ``sys.path`` before running the project's source scripts.
And, it ensures that any missing project dependencies are available in the
staging area, by downloading and installing them if necessary.

Last, but not least, the ``develop`` command invokes the ``build_ext -i``
command to ensure any C extensions in the project have been built and are
up-to-date, and the ``egg_info`` command to ensure the project's metadata is
updated (so that the runtime and wrappers know what the project's dependencies
are).  If you make any changes to the project's setup script or C extensions,
you should rerun the ``develop`` command against all relevant staging areas to
keep the project's scripts, metadata and extensions up-to-date.  Most other
kinds of changes to your project should not require any build operations or
rerunning ``develop``, but keep in mind that even minor changes to the setup
script (e.g. changing an entry point definition) require you to re-run the
``develop`` or ``test`` commands to keep the distribution updated.

Here are some of the options that the ``develop`` command accepts.  Note that
they affect the project's dependencies as well as the project itself, so if you
have dependencies that need to be installed and you use ``--exclude-scripts``
(for example), the dependencies' scripts will not be installed either!  For
this reason, you may want to use EasyInstall to install the project's
dependencies before using the ``develop`` command, if you need finer control
over the installation options for dependencies.

``--uninstall, -u``
    Un-deploy the current project.  You may use the ``--install-dir`` or ``-d``
    option to designate the staging area.  The created ``.egg-link`` file will
    be removed, if present and it is still pointing to the project directory.
    The project directory will be removed from ``easy-install.pth`` if the
    staging area is Python's ``site-packages`` directory.

    Note that this option currently does *not* uninstall script wrappers!  You
    must uninstall them yourself, or overwrite them by using EasyInstall to
    activate a different version of the package.  You can also avoid installing
    script wrappers in the first place, if you use the ``--exclude-scripts``
    (aka ``-x``) option when you run ``develop`` to deploy the project.

``--multi-version, -m``
    "Multi-version" mode. Specifying this option prevents ``develop`` from
    adding an ``easy-install.pth`` entry for the project(s) being deployed, and
    if an entry for any version of a project already exists, the entry will be
    removed upon successful deployment.  In multi-version mode, no specific
    version of the package is available for importing, unless you use
    ``pkg_resources.require()`` to put it on ``sys.path``, or you are running
    a wrapper script generated by ``setuptools`` or EasyInstall.  (In which
    case the wrapper script calls ``require()`` for you.)

    Note that if you install to a directory other than ``site-packages``,
    this option is automatically in effect, because ``.pth`` files can only be
    used in ``site-packages`` (at least in Python 2.3 and 2.4). So, if you use
    the ``--install-dir`` or ``-d`` option (or they are set via configuration
    file(s)) your project and its dependencies will be deployed in multi-
    version mode.

``--install-dir=DIR, -d DIR``
    Set the installation directory (staging area).  If this option is not
    directly specified on the command line or in a distutils configuration
    file, the distutils default installation location is used.  Normally, this
    will be the ``site-packages`` directory, but if you are using distutils
    configuration files, setting things like ``prefix`` or ``install_lib``,
    then those settings are taken into account when computing the default
    staging area.

``--script-dir=DIR, -s DIR``
    Set the script installation directory.  If you don't supply this option
    (via the command line or a configuration file), but you *have* supplied
    an ``--install-dir`` (via command line or config file), then this option
    defaults to the same directory, so that the scripts will be able to find
    their associated package installation.  Otherwise, this setting defaults
    to the location where the distutils would normally install scripts, taking
    any distutils configuration file settings into account.

``--exclude-scripts, -x``
    Don't deploy script wrappers.  This is useful if you don't want to disturb
    existing versions of the scripts in the staging area.

``--always-copy, -a``
    Copy all needed distributions to the staging area, even if they
    are already present in another directory on ``sys.path``.  By default, if
    a requirement can be met using a distribution that is already available in
    a directory on ``sys.path``, it will not be copied to the staging area.

``--egg-path=DIR``
    Force the generated ``.egg-link`` file to use a specified relative path
    to the source directory.  This can be useful in circumstances where your
    installation directory is being shared by code running under multiple
    platforms (e.g. Mac and Windows) which have different absolute locations
    for the code under development, but the same *relative* locations with
    respect to the installation directory.  If you use this option when
    installing, you must supply the same relative path when uninstalling.

In addition to the above options, the ``develop`` command also accepts all of
the same options accepted by ``easy_install``.  If you've configured any
``easy_install`` settings in your ``setup.cfg`` (or other distutils config
files), the ``develop`` command will use them as defaults, unless you override
them in a ``[develop]`` section or on the command line.


``easy_install`` - Find and install packages
============================================

This command runs the `EasyInstall tool
<easy_install.html>`_ for you.  It is exactly
equivalent to running the ``easy_install`` command.  All command line arguments
following this command are consumed and not processed further by the distutils,
so this must be the last command listed on the command line.  Please see
the EasyInstall documentation for the options reference and usage examples.
Normally, there is no reason to use this command via the command line, as you
can just use ``easy_install`` directly.  It's only listed here so that you know
it's a distutils command, which means that you can:

* create command aliases that use it,
* create distutils extensions that invoke it as a subcommand, and
* configure options for it in your ``setup.cfg`` or other distutils config
  files.


.. _egg_info:

``egg_info`` - Create egg metadata and set build tags
=====================================================

This command performs two operations: it updates a project's ``.egg-info``
metadata directory (used by the ``bdist_egg``, ``develop``, and ``test``
commands), and it allows you to temporarily change a project's version string,
to support "daily builds" or "snapshot" releases.  It is run automatically by
the ``sdist``, ``bdist_egg``, ``develop``, ``register``, and ``test`` commands
in order to update the project's metadata, but you can also specify it
explicitly in order to temporarily change the project's version string while
executing other commands.  (It also generates the``.egg-info/SOURCES.txt``
manifest file, which is used when you are building source distributions.)

In addition to writing the core egg metadata defined by ``setuptools`` and
required by ``pkg_resources``, this command can be extended to write other
metadata files as well, by defining entry points in the ``egg_info.writers``
group.  See the section on `Adding new EGG-INFO Files`_ below for more details.
Note that using additional metadata writers may require you to include a
``setup_requires`` argument to ``setup()`` in order to ensure that the desired
writers are available on ``sys.path``.


Release Tagging Options
-----------------------

The following options can be used to modify the project's version string for
all remaining commands on the setup command line.  The options are processed
in the order shown, so if you use more than one, the requested tags will be
added in the following order:

``--tag-build=NAME, -b NAME``
    Append NAME to the project's version string.  Due to the way setuptools
    processes "pre-release" version suffixes beginning with the letters "a"
    through "e" (like "alpha", "beta", and "candidate"), you will usually want
    to use a tag like ".build" or ".dev", as this will cause the version number
    to be considered *lower* than the project's default version.  (If you
    want to make the version number *higher* than the default version, you can
    always leave off --tag-build and then use one or both of the following
    options.)

    If you have a default build tag set in your ``setup.cfg``, you can suppress
    it on the command line using ``-b ""`` or ``--tag-build=""`` as an argument
    to the ``egg_info`` command.

``--tag-date, -d``
    Add a date stamp of the form "-YYYYMMDD" (e.g. "-20050528") to the
    project's version number.

``--no-date, -D``
    Don't include a date stamp in the version number.  This option is included
    so you can override a default setting in ``setup.cfg``.


(Note: Because these options modify the version number used for source and
binary distributions of your project, you should first make sure that you know
how the resulting version numbers will be interpreted by automated tools
like EasyInstall.  See the section above on `Specifying Your Project's
Version`_ for an explanation of pre- and post-release tags, as well as tips on
how to choose and verify a versioning scheme for your your project.)

For advanced uses, there is one other option that can be set, to change the
location of the project's ``.egg-info`` directory.  Commands that need to find
the project's source directory or metadata should get it from this setting:


Other ``egg_info`` Options
--------------------------

``--egg-base=SOURCEDIR, -e SOURCEDIR``
    Specify the directory that should contain the .egg-info directory.  This
    should normally be the root of your project's source tree (which is not
    necessarily the same as your project directory; some projects use a ``src``
    or ``lib`` subdirectory as the source root).  You should not normally need
    to specify this directory, as it is normally determined from the
    ``package_dir`` argument to the ``setup()`` function, if any.  If there is
    no ``package_dir`` set, this option defaults to the current directory.


``egg_info`` Examples
---------------------

Creating a dated "nightly build" snapshot egg::

    python setup.py egg_info --tag-date --tag-build=DEV bdist_egg

Creating and uploading a release with no version tags, even if some default
tags are specified in ``setup.cfg``::

    python setup.py egg_info -RDb "" sdist bdist_egg register upload

(Notice that ``egg_info`` must always appear on the command line *before* any
commands that you want the version changes to apply to.)


.. _install command:

``install`` - Run ``easy_install`` or old-style installation
============================================================

The setuptools ``install`` command is basically a shortcut to run the
``easy_install`` command on the current project.  However, for convenience
in creating "system packages" of setuptools-based projects, you can also
use this option:

``--single-version-externally-managed``
    This boolean option tells the ``install`` command to perform an "old style"
    installation, with the addition of an ``.egg-info`` directory so that the
    installed project will still have its metadata available and operate
    normally.  If you use this option, you *must* also specify the ``--root``
    or ``--record`` options (or both), because otherwise you will have no way
    to identify and remove the installed files.

This option is automatically in effect when ``install`` is invoked by another
distutils command, so that commands like ``bdist_wininst`` and ``bdist_rpm``
will create system packages of eggs.  It is also automatically in effect if
you specify the ``--root`` option.


``install_egg_info`` - Install an ``.egg-info`` directory in ``site-packages``
==============================================================================

Setuptools runs this command as part of ``install`` operations that use the
``--single-version-externally-managed`` options.  You should not invoke it
directly; it is documented here for completeness and so that distutils
extensions such as system package builders can make use of it.  This command
has only one option:

``--install-dir=DIR, -d DIR``
    The parent directory where the ``.egg-info`` directory will be placed.
    Defaults to the same as the ``--install-dir`` option specified for the
    ``install_lib`` command, which is usually the system ``site-packages``
    directory.

This command assumes that the ``egg_info`` command has been given valid options
via the command line or ``setup.cfg``, as it will invoke the ``egg_info``
command and use its options to locate the project's source ``.egg-info``
directory.


.. _rotate:

``rotate`` - Delete outdated distribution files
===============================================

As you develop new versions of your project, your distribution (``dist``)
directory will gradually fill up with older source and/or binary distribution
files.  The ``rotate`` command lets you automatically clean these up, keeping
only the N most-recently modified files matching a given pattern.

``--match=PATTERNLIST, -m PATTERNLIST``
    Comma-separated list of glob patterns to match.  This option is *required*.
    The project name and ``-*`` is prepended to the supplied patterns, in order
    to match only distributions belonging to the current project (in case you
    have a shared distribution directory for multiple projects).  Typically,
    you will use a glob pattern like ``.zip`` or ``.egg`` to match files of
    the specified type.  Note that each supplied pattern is treated as a
    distinct group of files for purposes of selecting files to delete.

``--keep=COUNT, -k COUNT``
    Number of matching distributions to keep.  For each group of files
    identified by a pattern specified with the ``--match`` option, delete all
    but the COUNT most-recently-modified files in that group.  This option is
    *required*.

``--dist-dir=DIR, -d DIR``
    Directory where the distributions are.  This defaults to the value of the
    ``bdist`` command's ``--dist-dir`` option, which will usually be the
    project's ``dist`` subdirectory.

**Example 1**: Delete all .tar.gz files from the distribution directory, except
for the 3 most recently modified ones::

    setup.py rotate --match=.tar.gz --keep=3

**Example 2**: Delete all Python 2.3 or Python 2.4 eggs from the distribution
directory, except the most recently modified one for each Python version::

    setup.py rotate --match=-py2.3*.egg,-py2.4*.egg --keep=1


.. _saveopts:

``saveopts`` - Save used options to a configuration file
========================================================

Finding and editing ``distutils`` configuration files can be a pain, especially
since you also have to translate the configuration options from command-line
form to the proper configuration file format.  You can avoid these hassles by
using the ``saveopts`` command.  Just add it to the command line to save the
options you used.  For example, this command builds the project using
the ``mingw32`` C compiler, then saves the --compiler setting as the default
for future builds (even those run implicitly by the ``install`` command)::

    setup.py build --compiler=mingw32 saveopts

The ``saveopts`` command saves all options for every command specified on the
command line to the project's local ``setup.cfg`` file, unless you use one of
the `configuration file options`_ to change where the options are saved.  For
example, this command does the same as above, but saves the compiler setting
to the site-wide (global) distutils configuration::

    setup.py build --compiler=mingw32 saveopts -g

Note that it doesn't matter where you place the ``saveopts`` command on the
command line; it will still save all the options specified for all commands.
For example, this is another valid way to spell the last example::

    setup.py saveopts -g build --compiler=mingw32

Note, however, that all of the commands specified are always run, regardless of
where ``saveopts`` is placed on the command line.


Configuration File Options
--------------------------

Normally, settings such as options and aliases are saved to the project's
local ``setup.cfg`` file.  But you can override this and save them to the
global or per-user configuration files, or to a manually-specified filename.

``--global-config, -g``
    Save settings to the global ``distutils.cfg`` file inside the ``distutils``
    package directory.  You must have write access to that directory to use
    this option.  You also can't combine this option with ``-u`` or ``-f``.

``--user-config, -u``
    Save settings to the current user's ``~/.pydistutils.cfg`` (POSIX) or
    ``$HOME/pydistutils.cfg`` (Windows) file.  You can't combine this option
    with ``-g`` or ``-f``.

``--filename=FILENAME, -f FILENAME``
    Save settings to the specified configuration file to use.  You can't
    combine this option with ``-g`` or ``-u``.  Note that if you specify a
    non-standard filename, the ``distutils`` and ``setuptools`` will not
    use the file's contents.  This option is mainly included for use in
    testing.

These options are used by other ``setuptools`` commands that modify
configuration files, such as the `alias`_ and `setopt`_ commands.


.. _setopt:

``setopt`` - Set a distutils or setuptools option in a config file
==================================================================

This command is mainly for use by scripts, but it can also be used as a quick
and dirty way to change a distutils configuration option without having to
remember what file the options are in and then open an editor.

**Example 1**.  Set the default C compiler to ``mingw32`` (using long option
names)::

    setup.py setopt --command=build --option=compiler --set-value=mingw32

**Example 2**.  Remove any setting for the distutils default package
installation directory (short option names)::

    setup.py setopt -c install -o install_lib -r


Options for the ``setopt`` command:

``--command=COMMAND, -c COMMAND``
    Command to set the option for.  This option is required.

``--option=OPTION, -o OPTION``
    The name of the option to set.  This option is required.

``--set-value=VALUE, -s VALUE``
    The value to set the option to.  Not needed if ``-r`` or ``--remove`` is
    set.

``--remove, -r``
    Remove (unset) the option, instead of setting it.

In addition to the above options, you may use any of the `configuration file
options`_ (listed under the `saveopts`_ command, above) to determine which
distutils configuration file the option will be added to (or removed from).


.. _test:

``test`` - Build package and run a unittest suite
=================================================

When doing test-driven development, or running automated builds that need
testing before they are deployed for downloading or use, it's often useful
to be able to run a project's unit tests without actually deploying the project
anywhere, even using the ``develop`` command.  The ``test`` command runs a
project's unit tests without actually deploying it, by temporarily putting the
project's source on ``sys.path``, after first running ``build_ext -i`` and
``egg_info`` to ensure that any C extensions and project metadata are
up-to-date.

To use this command, your project's tests must be wrapped in a ``unittest``
test suite by either a function, a ``TestCase`` class or method, or a module
or package containing ``TestCase`` classes.  If the named suite is a module,
and the module has an ``additional_tests()`` function, it is called and the
result (which must be a ``unittest.TestSuite``) is added to the tests to be
run.  If the named suite is a package, any submodules and subpackages are
recursively added to the overall test suite.  (Note: if your project specifies
a ``test_loader``, the rules for processing the chosen ``test_suite`` may
differ; see the `test_loader`_ documentation for more details.)

Note that many test systems including ``doctest`` support wrapping their
non-``unittest`` tests in ``TestSuite`` objects.  So, if you are using a test
package that does not support this, we suggest you encourage its developers to
implement test suite support, as this is a convenient and standard way to
aggregate a collection of tests to be run under a common test harness.

By default, tests will be run in the "verbose" mode of the ``unittest``
package's text test runner, but you can get the "quiet" mode (just dots) if
you supply the ``-q`` or ``--quiet`` option, either as a global option to
the setup script (e.g. ``setup.py -q test``) or as an option for the ``test``
command itself (e.g. ``setup.py test -q``).  There is one other option
available:

``--test-suite=NAME, -s NAME``
    Specify the test suite (or module, class, or method) to be run
    (e.g. ``some_module.test_suite``).  The default for this option can be
    set by giving a ``test_suite`` argument to the ``setup()`` function, e.g.::

        setup(
            # ...
            test_suite="my_package.tests.test_all"
        )

    If you did not set a ``test_suite`` in your ``setup()`` call, and do not
    provide a ``--test-suite`` option, an error will occur.


.. _upload:

``upload`` - Upload source and/or egg distributions to PyPI
===========================================================

The ``upload`` command is implemented and `documented
<https://docs.python.org/3.1/distutils/uploading.html>`_
in distutils.

Setuptools augments the ``upload`` command with support
for `keyring <https://pypi.python.org/pypi/keyring>`_,
allowing the password to be stored in a secure
location and not in plaintext in the .pypirc file. To use
keyring, first install keyring and set the password for
the relevant repository, e.g.::

    python -m keyring set <repository> <username>
    Password for '<username>' in '<repository>': ********

Then, in .pypirc, set the repository configuration as normal,
but omit the password. Thereafter, uploads will use the
password from the keyring.

New in 20.1: Added keyring support.


-----------------------------------------
Configuring setup() using setup.cfg files
-----------------------------------------

.. note:: New in 30.3.0 (8 Dec 2016).

.. important:: ``setup.py`` with ``setup()`` function call is still required even 
                if your configuration resides in ``setup.cfg``.

``Setuptools`` allows using configuration files (usually `setup.cfg`)
to define package’s metadata and other options which are normally supplied
to ``setup()`` function.

This approach not only allows automation scenarios, but also reduces
boilerplate code in some cases.

.. note::
    Implementation presents limited compatibility with distutils2-like
    ``setup.cfg`` sections (used by ``pbr`` and ``d2to1`` packages).

    Namely: only metadata related keys from ``metadata`` section are supported
    (except for ``description-file``); keys from ``files``, ``entry_points``
    and ``backwards_compat`` are not supported.


.. code-block:: ini

    [metadata]
    name = my_package
    version = attr: src.VERSION
    description = My package description
    long_description = file: README.rst
    keywords = one, two
    license = BSD 3-Clause License
    classifiers =
        Framework :: Django
        Programming Language :: Python :: 3
        Programming Language :: Python :: 3.5

    [options]
    zip_safe = False
    include_package_data = True
    packages = find:
    scripts =
      bin/first.py
      bin/second.py

    [options.package_data]
    * = *.txt, *.rst
    hello = *.msg

    [options.extras_require]
    pdf = ReportLab>=1.2; RXP
    rest = docutils>=0.3; pack ==1.1, ==1.3

    [options.packages.find]
    exclude =
        src.subpackage1
        src.subpackage2


Metadata and options could be set in sections with the same names.

* Keys are the same as keyword arguments one provides to ``setup()`` function.

* Complex values could be placed comma-separated or one per line
  in *dangling* sections. The following are the same:

  .. code-block:: ini

      [metadata]
      keywords = one, two

      [metadata]
      keywords =
        one
        two

* In some cases complex values could be provided in subsections for clarity.

* Some keys allow ``file:``, ``attr:`` and ``find:`` directives to cover
  common usecases.

* Unknown keys are ignored.


Specifying values
=================

Some values are treated as simple strings, some allow more logic.

Type names used below:

* ``str`` - simple string
* ``list-comma`` - dangling list or comma-separated values string
* ``list-semi`` - dangling list or semicolon-separated values string
* ``bool`` -  ``True`` is 1, yes, true
* ``dict`` - list-comma where keys from values are separated by =
* ``section`` - values could be read from a dedicated (sub)section


Special directives:

* ``attr:`` - value could be read from module attribute
* ``file:`` - value could be read from a file


.. note::
    ``file:`` directive is sandboxed and won't reach anything outside
    directory with ``setup.py``.


Metadata
--------

.. note::
    Aliases given below are supported for compatibility reasons,
    but not advised.

=================  =================  =====
Key                Aliases            Accepted value type
=================  =================  =====
name                                  str
version                               attr:, str
url                home-page          str
download_url       download-url       str
author                                str
author_email       author-email       str
maintainer                            str
maintainer_email   maintainer-email   str
classifiers        classifier         file:, list-comma
license                               file:, str
description        summary            file:, str
long_description   long-description   file:, str
keywords                              list-comma
platforms          platform           list-comma
provides                              list-comma
requires                              list-comma
obsoletes                             list-comma
=================  =================  =====

.. note::

    **version** - ``attr:`` supports callables; supports iterables;
    unsupported types are casted using ``str()``.


Options
-------

=======================  =====
Key                      Accepted value type
=======================  =====
zip_safe                 bool
setup_requires           list-semi
install_requires         list-semi
extras_require           section
python_requires          str
entry_points             file:, section
use_2to3                 bool
use_2to3_fixers          list-comma
use_2to3_exclude_fixers  list-comma
convert_2to3_doctests    list-comma
scripts                  list-comma
eager_resources          list-comma
dependency_links         list-comma
tests_require            list-semi
include_package_data     bool
packages                 find:, list-comma
package_dir              dict
package_data             section
exclude_package_data     section
namespace_packages       list-comma
py_modules               list-comma
=======================  =====

.. note::

    **packages** - ``find:`` directive can be further configured
    in a dedicated subsection `options.packages.find`. This subsection
    accepts the same keys as `setuptools.find` function:
    `where`, `include`, `exclude`.


Configuration API
=================

Some automation tools may wish to access data from a configuration file.

``Setuptools`` exposes ``read_configuration()`` function allowing
parsing ``metadata`` and ``options`` sections into a dictionary.


.. code-block:: python

    from setuptools.config import read_configuration

    conf_dict = read_configuration('/home/user/dev/package/setup.cfg')


By default ``read_configuration()`` will read only file provided
in the first argument. To include values from other configuration files
which could be in various places set `find_others` function argument
to ``True``.

If you have only a configuration file but not the whole package you can still
try to get data out of it with the help of `ignore_option_errors` function
argument. When it is set to ``True`` all options with errors possibly produced
by directives, such as ``attr:`` and others will be silently ignored.
As a consequence the resulting dictionary will include no such options.


--------------------------------
Extending and Reusing Setuptools
--------------------------------

Creating ``distutils`` Extensions
=================================

It can be hard to add new commands or setup arguments to the distutils.  But
the ``setuptools`` package makes it a bit easier, by allowing you to distribute
a distutils extension as a separate project, and then have projects that need
the extension just refer to it in their ``setup_requires`` argument.

With ``setuptools``, your distutils extension projects can hook in new
commands and ``setup()`` arguments just by defining "entry points".  These
are mappings from command or argument names to a specification of where to
import a handler from.  (See the section on `Dynamic Discovery of Services and
Plugins`_ above for some more background on entry points.)


Adding Commands
---------------

You can add new ``setup`` commands by defining entry points in the
``distutils.commands`` group.  For example, if you wanted to add a ``foo``
command, you might add something like this to your distutils extension
project's setup script::

    setup(
        # ...
        entry_points={
            "distutils.commands": [
                "foo = mypackage.some_module:foo",
            ],
        },
    )

(Assuming, of course, that the ``foo`` class in ``mypackage.some_module`` is
a ``setuptools.Command`` subclass.)

Once a project containing such entry points has been activated on ``sys.path``,
(e.g. by running "install" or "develop" with a site-packages installation
directory) the command(s) will be available to any ``setuptools``-based setup
scripts.  It is not necessary to use the ``--command-packages`` option or
to monkeypatch the ``distutils.command`` package to install your commands;
``setuptools`` automatically adds a wrapper to the distutils to search for
entry points in the active distributions on ``sys.path``.  In fact, this is
how setuptools' own commands are installed: the setuptools project's setup
script defines entry points for them!


Adding ``setup()`` Arguments
----------------------------

Sometimes, your commands may need additional arguments to the ``setup()``
call.  You can enable this by defining entry points in the
``distutils.setup_keywords`` group.  For example, if you wanted a ``setup()``
argument called ``bar_baz``, you might add something like this to your
distutils extension project's setup script::

    setup(
        # ...
        entry_points={
            "distutils.commands": [
                "foo = mypackage.some_module:foo",
            ],
            "distutils.setup_keywords": [
                "bar_baz = mypackage.some_module:validate_bar_baz",
            ],
        },
    )

The idea here is that the entry point defines a function that will be called
to validate the ``setup()`` argument, if it's supplied.  The ``Distribution``
object will have the initial value of the attribute set to ``None``, and the
validation function will only be called if the ``setup()`` call sets it to
a non-None value.  Here's an example validation function::

    def assert_bool(dist, attr, value):
        """Verify that value is True, False, 0, or 1"""
        if bool(value) != value:
            raise DistutilsSetupError(
                "%r must be a boolean value (got %r)" % (attr,value)
            )

Your function should accept three arguments: the ``Distribution`` object,
the attribute name, and the attribute value.  It should raise a
``DistutilsSetupError`` (from the ``distutils.errors`` module) if the argument
is invalid.  Remember, your function will only be called with non-None values,
and the default value of arguments defined this way is always None.  So, your
commands should always be prepared for the possibility that the attribute will
be ``None`` when they access it later.

If more than one active distribution defines an entry point for the same
``setup()`` argument, *all* of them will be called.  This allows multiple
distutils extensions to define a common argument, as long as they agree on
what values of that argument are valid.

Also note that as with commands, it is not necessary to subclass or monkeypatch
the distutils ``Distribution`` class in order to add your arguments; it is
sufficient to define the entry points in your extension, as long as any setup
script using your extension lists your project in its ``setup_requires``
argument.


Adding new EGG-INFO Files
-------------------------

Some extensible applications or frameworks may want to allow third parties to
develop plugins with application or framework-specific metadata included in
the plugins' EGG-INFO directory, for easy access via the ``pkg_resources``
metadata API.  The easiest way to allow this is to create a distutils extension
to be used from the plugin projects' setup scripts (via ``setup_requires``)
that defines a new setup keyword, and then uses that data to write an EGG-INFO
file when the ``egg_info`` command is run.

The ``egg_info`` command looks for extension points in an ``egg_info.writers``
group, and calls them to write the files.  Here's a simple example of a
distutils extension defining a setup argument ``foo_bar``, which is a list of
lines that will be written to ``foo_bar.txt`` in the EGG-INFO directory of any
project that uses the argument::

    setup(
        # ...
        entry_points={
            "distutils.setup_keywords": [
                "foo_bar = setuptools.dist:assert_string_list",
            ],
            "egg_info.writers": [
                "foo_bar.txt = setuptools.command.egg_info:write_arg",
            ],
        },
    )

This simple example makes use of two utility functions defined by setuptools
for its own use: a routine to validate that a setup keyword is a sequence of
strings, and another one that looks up a setup argument and writes it to
a file.  Here's what the writer utility looks like::

    def write_arg(cmd, basename, filename):
        argname = os.path.splitext(basename)[0]
        value = getattr(cmd.distribution, argname, None)
        if value is not None:
            value = '\n'.join(value) + '\n'
        cmd.write_or_delete_file(argname, filename, value)

As you can see, ``egg_info.writers`` entry points must be a function taking
three arguments: a ``egg_info`` command instance, the basename of the file to
write (e.g. ``foo_bar.txt``), and the actual full filename that should be
written to.

In general, writer functions should honor the command object's ``dry_run``
setting when writing files, and use the ``distutils.log`` object to do any
console output.  The easiest way to conform to this requirement is to use
the ``cmd`` object's ``write_file()``, ``delete_file()``, and
``write_or_delete_file()`` methods exclusively for your file operations.  See
those methods' docstrings for more details.


Adding Support for Revision Control Systems
-------------------------------------------------

If the files you want to include in the source distribution are tracked using
Git, Mercurial or SVN, you can use the following packages to achieve that:

- Git and Mercurial: `setuptools_scm <https://pypi.python.org/pypi/setuptools_scm>`_
- SVN: `setuptools_svn <https://pypi.python.org/pypi/setuptools_svn>`_

If you would like to create a plugin for ``setuptools`` to find files tracked
by another revision control system, you can do so by adding an entry point to
the ``setuptools.file_finders`` group.  The entry point should be a function
accepting a single directory name, and should yield all the filenames within
that directory (and any subdirectories thereof) that are under revision
control.

For example, if you were going to create a plugin for a revision control system
called "foobar", you would write a function something like this:

.. code-block:: python

    def find_files_for_foobar(dirname):
        # loop to yield paths that start with `dirname`

And you would register it in a setup script using something like this::

    entry_points={
        "setuptools.file_finders": [
            "foobar = my_foobar_module:find_files_for_foobar",
        ]
    }

Then, anyone who wants to use your plugin can simply install it, and their
local setuptools installation will be able to find the necessary files.

It is not necessary to distribute source control plugins with projects that
simply use the other source control system, or to specify the plugins in
``setup_requires``.  When you create a source distribution with the ``sdist``
command, setuptools automatically records what files were found in the
``SOURCES.txt`` file.  That way, recipients of source distributions don't need
to have revision control at all.  However, if someone is working on a package
by checking out with that system, they will need the same plugin(s) that the
original author is using.

A few important points for writing revision control file finders:

* Your finder function MUST return relative paths, created by appending to the
  passed-in directory name.  Absolute paths are NOT allowed, nor are relative
  paths that reference a parent directory of the passed-in directory.

* Your finder function MUST accept an empty string as the directory name,
  meaning the current directory.  You MUST NOT convert this to a dot; just
  yield relative paths.  So, yielding a subdirectory named ``some/dir`` under
  the current directory should NOT be rendered as ``./some/dir`` or
  ``/somewhere/some/dir``, but *always* as simply ``some/dir``

* Your finder function SHOULD NOT raise any errors, and SHOULD deal gracefully
  with the absence of needed programs (i.e., ones belonging to the revision
  control system itself.  It *may*, however, use ``distutils.log.warn()`` to
  inform the user of the missing program(s).


Subclassing ``Command``
-----------------------

Sorry, this section isn't written yet, and neither is a lot of what's below
this point.

XXX


Reusing ``setuptools`` Code
===========================

``ez_setup``
------------

XXX


``setuptools.archive_util``
---------------------------

XXX


``setuptools.sandbox``
----------------------

XXX


``setuptools.package_index``
----------------------------

XXX


Mailing List and Bug Tracker
============================

Please use the `distutils-sig mailing list`_ for questions and discussion about
setuptools, and the `setuptools bug tracker`_ ONLY for issues you have
confirmed via the list are actual bugs, and which you have reduced to a minimal
set of steps to reproduce.

.. _distutils-sig mailing list: http://mail.python.org/pipermail/distutils-sig/
.. _setuptools bug tracker: https://github.com/pypa/setuptools/

# Created by: thierry@pompo.net
# $FreeBSD$

PORTNAME=	code_saturne
PORTVERSION=	6.1.0
PORTREVISION=	1
DISTVERSIONPREFIX=	v
CATEGORIES=	science parallel

MAINTAINER=	thierry@FreeBSD.org
COMMENT=	EDF's general purpose Computational Fluid Dynamics (CFD) software

LICENSE=	GPLv2
LICENSE_FILE=	${WRKSRC}/COPYING

BUILD_DEPENDS=	gsed:textproc/gsed
LIB_DEPENDS=	libsysinfo.so:devel/libsysinfo	\
		libcgns.so:science/cgnslib	\
		libhdf5.so:science/hdf5		\
		libmedC.so:french/med
RUN_DEPENDS=	${PYTHON_PKGNAMEPREFIX}matplotlib>0:math/py-matplotlib@${PY_FLAVOR}	\
		xmgrace:math/grace	\
		bash:shells/bash	\
		xterm:x11/xterm

USE_GITHUB=	yes
GH_ACCOUNT=	code-saturne
GH_TAGNAME=	89e7bbf	# Some fixes are needed for atmospheric aerosols

USES=		autoreconf bison fortran gettext gnome gmake iconv libtool	\
		pyqt:5 python:3.6+ shebangfix

USE_GNOME=	libxml2
USE_PYQT=	core gui sip widgets xml_build
SHEBANG_FILES=	install_saturne.py tests/unittests.py bin/*.py build-aux/*.py
CPPFLAGS+=	-I${LOCALBASE}/include -I${LOCALBASE}/include/libxml2
GNU_CONFIGURE=	yes
CONFIGURE_ARGS=	--without-salome --without-salome-yacs --without-salome-kernel	\
		--without-salome-gui
CONFIGURE_ENV=	ac_cv_path_SED=${LOCALBASE}/bin/gsed
USE_LDCONFIG=	yes
BINARY_ALIAS=	python=${PYTHON_CMD} \
		pyrcc5=pyrcc5-${PYTHON_VER} \
		pyuic5=pyuic5-${PYTHON_VER}

OPTIONS_DEFINE=		DOCS EXAMPLES BATCH BLAS MPI
OPTIONS_DEFAULT=	BATCH BLAS MPI METIS
OPTIONS_RADIO=		METIS
OPTIONS_RADIO_METIS=	METIS PARMETIS
BLAS_DESC=		Support of OpenBLAS
BLAS_USES=		blaslapack
BLAS_CONFIGURE_ON=	--with-blas=yes --with-blas-lib=${LOCALBASE}/lib --with-blas-libs=-lopenblas
BLAS_LIB_DEPENDS=	libopenblas.so:math/openblas
BATCH_DESC=		Use Torque to submit batches
BATCH_RUN_DEPENDS=	pbs-config:sysutils/torque
MPI_LIB_DEPENDS=	libmpi.so:net/mpich
MPI_CONFIGURE_WITH=	mpi
METIS_DESC=		Support of METIS
METIS_LIB_DEPENDS=	libmetis.so:math/metis
METIS_CONFIGURE_ON=	--with-metis-include="${LOCALBASE}/include"
PARMETIS_DESC=		Support of ParMETIS
PARMETIS_LIB_DEPENDS=	libparmetis.so:math/parmetis
PARMETIS_CONFIGURE_ON=	--with-metis-include="${LOCALBASE}/include/parmetis"	\
			--with-metis-lib="${LOCALBASE}/lib/parmetis"
PARMETIS_CONFIGURE_ENV=	LIBS=-lexecinfo

TEST_TARGET=	check

.if defined(PACKAGE_BUILDING)
TERM=		vt100	# Force for pointyhat to override su
.else
TERM?=		vt100	# Default value needed for tput in jail or tinderbox
.endif

.include <bsd.port.options.mk>

.if ${ARCH} == "i386"
# /wrkdirs/usr/ports/science/code_saturne/work/code_saturne-5.3.0/src/apps/.libs
# /libsaturne.so:
# undefined reference to `__atomic_compare_exchange'
CONFIGURE_ARGS+=	--disable-openmp
.endif

.if ${PORT_OPTIONS:MDOCS}
USE_TEX=	tex:build
BUILD_DEPENDS+=	fig2dev:print/transfig		\
		doxygen:devel/doxygen		\
		dot:graphics/graphviz		\
		${PY_SPHINX}
BINARY_ALIAS+=	sphinx-build=sphinx-build-${PYTHON_VER}
ALL_TARGET=	all pdf
DOCS=		AUTHORS ChangeLog README
.endif

pre-configure:
	${FLEX} -o ${WRKSRC}/src/mei/mei_scanner.c ${WRKSRC}/src/mei/mei_scanner.l
	${LOCALBASE}/bin/bison --defines=${WRKSRC}/src/mei/mei_parser.h	\
		-o ${WRKSRC}/src/mei/mei_parser.c ${WRKSRC}/src/mei/mei_parser.y
	${REINPLACE_CMD} -e 's|/usr/bin/env python|${PYTHON_CMD}|'	\
		${WRKSRC}/Makefile.am

post-install-DOCS-on:
	@${MKDIR} ${STAGEDIR}${DOCSDIR}
	${INSTALL_DATA} ${DOCS:C|^|${WRKSRC}/|} ${STAGEDIR}${DOCSDIR}

post-install-EXAMPLES-on:
	@${MKDIR} ${STAGEDIR}${EXAMPLESDIR}
	cd ${WRKSRC}/examples && ${COPYTREE_SHARE} . ${STAGEDIR}${EXAMPLESDIR}

.include <bsd.port.mk>

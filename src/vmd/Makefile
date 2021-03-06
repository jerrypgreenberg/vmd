# @Copyright@
# 
# 				Rocks(r)
# 		         www.rocksclusters.org
# 		         version 5.6 (Emerald Boa)
# 		         version 6.1 (Emerald Boa)
# 
# Copyright (c) 2000 - 2013 The Regents of the University of California.
# All rights reserved.	
# 
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are
# met:
# 
# 1. Redistributions of source code must retain the above copyright
# notice, this list of conditions and the following disclaimer.
# 
# 2. Redistributions in binary form must reproduce the above copyright
# notice unmodified and in its entirety, this list of conditions and the
# following disclaimer in the documentation and/or other materials provided 
# with the distribution.
# 
# 3. All advertising and press materials, printed or electronic, mentioning
# features or use of this software must display the following acknowledgement: 
# 
# 	"This product includes software developed by the Rocks(r)
# 	Cluster Group at the San Diego Supercomputer Center at the
# 	University of California, San Diego and its contributors."
# 
# 4. Except as permitted for the purposes of acknowledgment in paragraph 3,
# neither the name or logo of this software nor the names of its
# authors may be used to endorse or promote products derived from this
# software without specific prior written permission.  The name of the
# software includes the following terms, and any derivatives thereof:
# "Rocks", "Rocks Clusters", and "Avalanche Installer".  For licensing of 
# the associated name, interested parties should contact Technology 
# Transfer & Intellectual Property Services, University of California, 
# San Diego, 9500 Gilman Drive, Mail Code 0910, La Jolla, CA 92093-0910, 
# Ph: (858) 534-5815, FAX: (858) 534-7345, E-MAIL:invent@ucsd.edu
# 
# THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS ``AS IS''
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
# THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE REGENTS OR CONTRIBUTORS
# BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR
# BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
# WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE
# OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN
# IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
# 
# @Copyright@
#
# $Log$

REDHAT.ROOT = $(CURDIR)/../../

-include $(ROCKSROOT)/etc/Rules.mk
include Rules.mk
include pull.mk
VMD_SOURCE=$(VMD_NAME)-$(VMD_VERSION)
TACHYON_SOURCE=$(TACHYON_NAME)-$(TACHYON_VERSION)
ACTC_SOURCE=$(ACTC_NAME)-$(ACTC_VERSION)
FLTK_SOURCE=$(FLTK_NAME)-$(FLTK_VERSION)

PKGROOT=/opt/vmd

build: $(SRC_DIRS)
	(\
            . /etc/profile.d/modules.sh; \
	    module load cmake; \
            module load python; \
            module load scipy; \
            module load netcdf; \
            export PYTHON_INCLUDE_DIR=/opt/python/include/python2.7; \
            export PYTHON_LIBRARY_DIR=/opt/python/lib; \
            export NUMPY_INCLUDE_DIR=/opt/scipy/lib/python2.7/site-packages/numpy/core/include; \
            patch $(VMD_SOURCE)/Makefile patch-files/Makefile.patch; \
            patch $(FLTK_SOURCE)/Makefile patch-files/Makefile.fltk.patch; \
            patch plugins/Make-arch patch-files/Make-arch.patch; \
            cd $(TACHYON_NAME)/unix; \
            make linux-icc-thr-ogl; \
            mkdir -p ../../$(VMD_SOURCE)/lib/$(TACHYON_NAME)/lib;\
            mkdir ../../$(VMD_SOURCE)/lib/$(TACHYON_NAME)/include;\
            cp ../compile/linux-icc-thr-ogl/libtachyon.a ../../$(VMD_SOURCE)/lib/$(TACHYON_NAME)/lib;\
            cd ../../$(ACTC_SOURCE); \
            export CC=icc; \
            make; \
            mkdir -p ../$(VMD_SOURCE)/lib/actc/lib; \
            mkdir ../$(VMD_SOURCE)/lib/actc/include; \
            cp *.a  ../$(VMD_SOURCE)/lib/actc/lib; \
            cp *.h  ../$(VMD_SOURCE)/lib/actc/include; \
            cd ../$(FLTK_SOURCE); \
            perl -pi -e "s!ROLLINSTALL!$(CURDIR)/$(VMD_SOURCE)/lib/fltk!" Makefile; \
            make; \
            make install; \
            cd ../$(VMD_SOURCE);\
            cd ../plugins; \
            export PLUGINDIR=$(REDHAT.ROOT)/src/build-plugins; \
            make LINUXAMD64; \
            make distrib; \
            cd ../$(VMD_SOURCE); \
            ln -s $(REDHAT.ROOT)/src/build-plugins  plugins; \
            patch configure $(REDHAT.ROOT)/src/vmd/patch-files/configure.patch; \
	    perl -pi -e "s!ROLLNETCDF!$${NETCDFHOME}!" configure; \
            make linux.amd64.opengl; \
            cp ../$(TACHYON_NAME)/src/*.h lib/$(TACHYON_NAME)/include;\
            cd src; \
	    perl -pi -e "s!/usr/local/bin/gzip!/bin/gzip!" Makefile; \
	    perl -pi -e "s!/usr/local!$(PKGROOT)!g" Makefile; \
            make \
	)
	@echo "::: Package $(NAME) build complete :::"
	
install::
	mkdir -p $(ROOT)/$(PKGROOT)
	mkdir -p $(PKGROOT)
	mkdir -p $(REDHAT.ROOT)/src/build-vmd-modules
	mount --bind $(ROOT)/$(PKGROOT) $(PKGROOT)
	(							\
            cd $(VMD_SOURCE)/src; \
            make install; \
            rm -f $(ROOT)/$(PKGROOT)/lib/vmd/plugins; \
            mkdir $(ROOT)/$(PKGROOT)/lib/vmd/plugins; \
            cp -r $(REDHAT.ROOT)/src/build-plugins/*  $(ROOT)/$(PKGROOT)/lib/vmd/plugins \
	)
	cat vmd.module | perl -pi -e "s!INSTALLROOT!$(PKGROOT)!g;s!MODULEROOT!/opt/modulefiles/applications/vmd!g;s!VERSION!$(VERSION)!g" \
          > $(REDHAT.ROOT)/src/build-vmd-modules/$(NAME).module; \

	@echo "::: Package $(NAME) install complete :::"

clean::
	rm -rf $(VMD_SOURCE) $(TACHYON_NAME) plugins $(ACTC_SOURCE) $(FLTK_SOURCE)
	-umount $(PKGROOT)

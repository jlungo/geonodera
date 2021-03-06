FROM python:2.7.14

RUN mkdir -p /usr/src/{app,geonode}

WORKDIR /usr/src/app

# This section is borrowed from the official Django image but adds GDAL and others
RUN apt-get update && apt-get install -y \
		gcc \
		gettext \
		postgresql-client libpq-dev \
		sqlite3 \
		python-gdal python-psycopg2 \
		python-imaging python-lxml \
		python-dev libgdal-dev \
        python-ldap \
        libmemcached-dev libsasl2-dev zlib1g-dev \
	--no-install-recommends && rm -rf /var/lib/apt/lists/*

# Upgrade pip
RUN pip install --upgrade pip

# To understand the next section (the need for requirements.txt and setup.py)
# Please read: https://packaging.python.org/requirements/

# python-gdal does not seem to work, let's install manually the version that is
# compatible with the provided libgdal-dev
RUN pip install GDAL==1.10 --global-option=build_ext --global-option="-I/usr/include/gdal"

# Install shallow clone of geonode branch
RUN git clone --depth=1 git://github.com/GeoNode/geonode.git --branch master /usr/src/geonode
COPY local_settings.py /usr/src/geonode/geonode/local_settings.py
RUN cd /usr/src/geonode/; pip install --upgrade --no-cache-dir -r requirements.txt; pip install --upgrade -e .

## There is a diff in tasks.py in the source and in the docker version
COPY entrypoint.sh /usr/src/app/entrypoint.sh
COPY tasks.py /usr/src/app/tasks.py
COPY uwsgi.ini /usr/src/app/uwsgi.ini
COPY wait-for-databases.sh /usr/bin/wait-for-databases

RUN chmod +x /usr/src/app/tasks.py \
    && chmod +x /usr/src/app/entrypoint.sh \
	&& chmod +x /usr/src/app/uwsgi.ini \
	&& chmod +x /usr/bin/wait-for-databases

EXPOSE 8000

ENTRYPOINT ["/usr/src/app/entrypoint.sh"]
CMD ["uwsgi", "--ini", "/usr/src/app/uwsgi.ini"]


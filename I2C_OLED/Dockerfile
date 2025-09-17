ARG BUILD_FROM
FROM $BUILD_FROM

# Install requirements for add-on
RUN apk add --no-cache \
  git \
  python3 \
  py3-pip \
  py3-pillow \
  py3-smbus

RUN rm -Rf /I2C_OLED
RUN git clone -b main --single-branch --depth 1 https://github.com/NickDub/rpi_i2c_oled.git /I2C_OLED

COPY run.sh /
WORKDIR /data
RUN chmod a+x /run.sh
CMD [ "/run.sh" ]

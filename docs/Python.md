---
### Tracking learning পাইথন
---

## পাইথন

Python started as a scripting language. Now it is a programming language on its own. Though Python uses C for speedy execution of its' modules.

### শেখার পয়েন্ট সমূহ

#### প্রোগ্রামিং ভাষা
- [x] Interpretted programming language
- [x] Sequence/Control of flow (line by line and indentation)
- [x] Data and data types
- [x] Operators
- [x] Containers
- [x] Iteration
- [ ] Selection
- [x] Functions (অসম্পূর্ণ)
- [ ] Object Oriented programming
- [ ] Doing projects

#### মডিউল সমূহ
- [x] Tk/Tcl (অসম্পূর্ণ)
- [ ] WxPython
- [ ] 
#### প্রজেক্ট সমূহ
- [x] `Hello, world!` in GUI.
- [ ]
- [ ]
- [ ]

***

### INSTALLATION
Download installer or zip. And install it to the location you like or the default location.

When using the zip environment variables will not be set. So Python will only be available in its installation directory. Setting environment variable will solve this.

> Create System/User variable: `PYTHON_HOME` and write value as the install directory of Python i.e. C:\Python38-32.<br>
Append the following text `;%PYTHON_HOME%\;%PYTHON_HOME%\Scripts\` to the end of the System/User variable named `Path`.

Additionally for the same effect, environment variable for PiP might become necessary.

> *Note:* If there is a problem with installing Python with default settings, try installing it as current user.

***

### EXTRA
Install a few useful Python modules.

<details>
  <summery>পিপ আপগ্রেড</summery>

  #### Upgrade PiP
  python -m pip install --upgrade pip
</details>

<details>
  <summery>সেটাপটুল্‌স ইনস্টল ও আপগ্রেড</summery>

  #### Install Setuptools
  pip -m install setuptools

  #### Upgrade Setuptools
  pip -m install setuptools --upgrade setuptools
</details>

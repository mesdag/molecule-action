# Molecule action

A GitHub action to tests your [Ansible](https://www.ansible.com/) role using [Molecule](https://molecule.readthedocs.io/en/stable/) and [Podman](https://podman.io/).

## Requirements

This action can work with Molecule scenarios that use the [`podman`](https://molecule.readthedocs.io/en/latest/configuration.html#driver) driver.

This action expects the following (default Ansible role) structure:
```
.
├── defaults
│   └── main.yml
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── molecule
│   └── default
│       ├── molecule.yml
│       ├── playbook.yml
│       └── prepare.yml
├── requirements.yml
├── tasks
│   └── main.yml
├── tox.ini # OPTIONAL
└── vars
    └── main.yml
```

If you are missing the `molecule` directory, you can try to run `molecule init scenario -r <role> -d podman`.

When `tox.ini` is found, [tox](https://tox.readthedocs.io/en/latest/) is used to test the role. Tox will install all dependecies found in `tox.ini` itself, meaning `tox.ini` determines the version of [molecule](https://molecule.readthedocs.io/en/latest/) that is used.

## Inputs

### `namespace`

The Quay.io namespace where the image is in. Default `"ucomesdag"`.

### `image`

The image you want to run on. Default `"fedora"`.

### `tag`

The tag of the container image to use. Default `"latest"`.

### `options`

The [options to pass to `tox`](https://tox.readthedocs.io/en/latest/config.html#tox). For example `parallel`. Default `""`. (empty)

### `command`

The molecule command to use. For example `lint`. Default `"test"`.

### `scenario`

The molecule scenario to run. Default `"default"`

## Example usage

Here is a default configuration that tests your role on `namespace: ucomesdag`, `image: fedora`, `tag: latest`.

```yaml
---
on:
  - push

jobs:
  build:
    runs-on: ubuntu-20.04
    steps:
      - name: checkout
        uses: actions/checkout@v3
        with:
          path: "${{ github.repository }}"
      - name: molecule
        uses: ucomesdag/molecule-action@main
```

NOTE: the `checkout` action needs to place the file in `${{ github.repository }}` in order for Molecule to find your role.

If you want to test your role against multiple distributions, you can use this pattern:

```yaml
---
name: CI

on:
  - push

jobs:
  lint:
    runs-on: ubuntu-20.04
    steps:
      - name: checkout
        uses: actions/checkout@v3
        with:
          path: "${{ github.repository }}"
      - name: molecule
        uses: ucomesdag/molecule-action@main
        with:
          command: lint
  test:
    needs:
      - lint
    runs-on: ubuntu-20.04
    strategy:
      matrix:
        image:
          - alpine
          - amazonlinux
          - debian
          - centos
          - fedora
          - opensuse
          - ubuntu
    steps:
      - name: checkout
        uses: actions/checkout@v3
        with:
          path: "${{ github.repository }}"
      - name: molecule
        uses: ucomesdag/molecule-action@main
        with:
          image: "${{ matrix.image }}"
          options: parallel
          scenario: my_specific_scenario
```

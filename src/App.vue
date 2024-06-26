<template>
  <a
    class="github-fork-ribbon"
    href="https://github.com/kritish-dhaubanjar/cidr-subnet-visualization"
    data-ribbon="Fork me on GitHub"
    title="Fork me on GitHub"
    >Fork me on GitHub</a
  >

  <div class="container my-5">
    <div class="row">
      <div class="col-12">
        <h1>IPv4 CIDR Notation</h1>

        <p>
          A system called
          <a
            target="_blank"
            href="https://www.digitalocean.com/community/tutorials/understanding-ip-addresses-subnets-and-cidr-notation-for-networking"
          >
            Classless Inter-Domain Routing,</a
          >
          or CIDR, was developed as an alternative to traditional subnetting.
          The idea is that you can add a specification in the IP address itself
          as to the number of significant bits that make up the routing or
          networking portion.
          <br />
          <br />
          For example, we could express the idea that the IP address
          <span class="fw-bold">192.168.0.15</span> is associated with the
          netmask <span class="fw-bold">255.255.255.0</span> by using the CIDR
          notation of <span class="fw-bold">192.168.0.15/24</span>. This means
          that the first 24 bits of the IP address given are considered
          significant for the network routing.
        </p>

        <em
          >CIDR allows us more control over addressing continuous blocks of IP
          addresses. This is much more useful than the subnetting using IPv4
          Addresses Classes and Reserved Ranges.</em
        >
      </div>
    </div>

    <div class="row mt-5 mb-4">
      <div
        class="col-12 input-group justify-content-center position-relative ip-address"
      >
        <template v-for="(_, i) in ipv4" :key="i">
          <input
            :value="ipv4[i]"
            @paste="handler"
            @keypress="validate"
            @input="(e) => setIPv4Segment(e, i)"
            min="0"
            max="255"
            type="number"
            class="form-control text-center"
          />
          <h1 v-show="i < 3" class="px-3">.</h1>
        </template>

        <h1 class="px-3">/</h1>
        <input
          :value="cidr"
          @input="setCIDR"
          @paste="handler"
          @keypress="validate"
          min="0"
          max="32"
          type="number"
          class="form-control text-center"
        />
      </div>
    </div>

    <div class="row">
      <div class="col-12">
        <h4 class="mt-4 mb-3">IPv4 Address</h4>
      </div>

      <div class="col-3 segment" v-for="(mask, i) in ipv4" :key="i">
        <Segment :position="i" :mask="mask" :cidr="cidr" />
      </div>

      <div class="col-12">
        <h4 class="mt-4 mb-3">Subnet Mask</h4>
      </div>

      <div
        class="col-3 segment"
        v-for="(mask, i) in subnet.subnetMask"
        :key="i"
      >
        <Segment :position="i" :mask="mask" :cidr="cidr" />
      </div>
    </div>

    <div class="row">
      <div class="col-12">
        <h4 class="mt-4">Subnet</h4>

        <table class="table">
          <tbody>
            <tr>
              <td scope="col">Total IPv4</td>
              <td scope="col">
                <code>{{ subnet.count }}</code>
              </td>
            </tr>
            <tr>
              <td scope="row">Subnet Mask</td>
              <td>
                <code>{{ subnet.subnetMask.join(".") }}</code>
              </td>
            </tr>
            <tr>
              <td scope="row">Wildcard Mask</td>
              <td>
                <code>{{ subnet.wildcardMask.join(".") }}</code>
              </td>
            </tr>
            <tr>
              <td scope="row">Network</td>
              <td>
                <code>{{ subnet.networkAddress.join(".") }}/{{ cidr }}</code>
              </td>
            </tr>
            <tr>
              <td scope="row">Broadcast Address</td>
              <td>
                <code>{{ subnet.broadcastAddress.join(".") }}</code>
              </td>
            </tr>
            <tr>
              <td scope="row">Host Addresses</td>
              <td>
                <code
                  >{{ subnet.hostAddresses.start.join(".") }} -
                  {{ subnet.hostAddresses.end.join(".") }}</code
                >
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <p class="mt-5 text-center">
      <em
        >* For routing mask values &gt;= 31, first and last IPs are network and
        broadcast addresses and are unusable.</em
      >
      <br />
      <span class="small text-end">
        Inspired from
        <a href="https://github.com/yuvadm/cidr.xyz" target="_blank"
          >https://cidr.xyz</a
        >.
      </span>
    </p>
  </div>
</template>

<script setup lang="ts">
/* eslint-disable @typescript-eslint/ban-ts-comment */

import { ref } from "@vue/reactivity";
import { nextTick, watch } from "@vue/runtime-core";

// @ts-ignore
import { computeSubnet } from "compute-subnet";

// @ts-ignore
import Segment from "./components/Segment.vue";

const NUMBERS = /[0-9/]+/;
const IPV4 =
  /^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\/?$/;
const IPV4_CIDR =
  /^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\/([0-9]|1[0-9]|2[0-9]|3[0-2])$/;

const cidr = ref(23);
const ipv4 = ref([192, 168, 100, 128]);
const subnet = ref(computeSubnet(ipv4.value, cidr.value));

function validate(e: KeyboardEvent) {
  if (!NUMBERS.test(e.key)) {
    e.preventDefault();
  }
}

function setCIDR(e: Event) {
  let valid = cidr.value;
  const data = parseInt((e.target as HTMLInputElement).value);

  if (data >= 0 && data <= 32) {
    valid = data;
  } else if (isNaN(data)) {
    valid = 0;
  }

  cidr.value = valid;
  subnet.value = computeSubnet(ipv4.value, cidr.value);
}

function handler(e: ClipboardEvent) {
  const clipboard = e?.clipboardData?.getData("text") ?? "";

  const data: {
    ipv4: number[];
    cidr: number;
  } = {
    ipv4: [192, 168, 100, 128],
    cidr: cidr.value,
  };

  if (!IPV4.test(clipboard) && !IPV4_CIDR.test(clipboard)) {
    return;
  }

  if (IPV4.test(clipboard)) {
    data.ipv4 = clipboard
      .replace(/\//g, "")
      .split(".")
      .map((e) => parseInt(e));
    data.cidr = cidr.value;
  }

  if (IPV4_CIDR.test(clipboard)) {
    const [IP, CIDR] = clipboard.split("/");
    const value = IP.split(".").map((e) => parseInt(e));

    data.ipv4 = value;
    data.cidr = parseInt(CIDR);
  }

  nextTick(() => {
    ipv4.value = data.ipv4;
    cidr.value = data.cidr;
    subnet.value = computeSubnet(data.ipv4, data.cidr);
  });
}

function setIPv4Segment(e: Event, index: number) {
  const valid = [...ipv4.value];
  const data = parseInt((e.target as HTMLInputElement).value);

  if (data >= 0 && data <= 255) {
    valid[index] = data;
  } else if (isNaN(data)) {
    valid[index] = 0;
  }

  ipv4.value = valid;
  subnet.value = computeSubnet(ipv4.value, cidr.value);
}
</script>

<style lang="scss">
@import "bootstrap";
@import "./assets/global.scss";
</style>

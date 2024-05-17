<script setup>
import { ref } from "vue";
import * as XLSX from "xlsx";
import {
  Table,
  TableBody,
  TableCaption,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

const dataLoaded = ref(false);
const points = ref([]);

const handleFileChange = (event) => {
  const file = event.target.files[0];
  const reader = new FileReader();
  reader.onload = (e) => {
    const data = new Uint8Array(e.target.result);
    const workbook = XLSX.read(data, { type: "array" });
    const sheetName = workbook.SheetNames[0];
    const worksheet = workbook.Sheets[sheetName];
    const parsedData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
    const pointsData = parsedData.slice(1).map((row) => ({
      tahun: row[1],
      kecamatan: row[2],
      nama: row[3],
      x: row[4],
      y: row[5],
      z: row[6],
    }));
    console.log(parsedData);
    points.value = pointsData;
    dataLoaded.value = true;
  };
  reader.readAsArrayBuffer(file);
};

const c1 = (point) => {
  const J = 324;
  const K = 354;
  const L = 0;
  return Math.sqrt(
    Math.pow(point.x - J, 2) +
      Math.pow(point.y - K, 2) +
      Math.pow(point.z - L, 2)
  );
};

function c2(point) {
  var J_init = 178;
  var K_init = 230;
  var L_init = 0;
  var distance = Math.sqrt(
    Math.pow(point.x - J_init, 2) +
      Math.pow(point.y - K_init, 2) +
      Math.pow(point.z - L_init, 2)
  );
  return distance;
}

function c3(point) {
  var J_init = 315;
  var K_init = 341;
  var L_init = 14;
  var distance = Math.sqrt(
    Math.pow(point.x - J_init, 2) +
      Math.pow(point.y - K_init, 2) +
      Math.pow(point.z - L_init, 2)
  );
  return distance;
}

function kedekatan(O5, P5, Q5) {
  let tmp = Math.min(O5, P5, Q5);
  let stat = "Tinggi (Bagus)";
  if (tmp == O5) stat = "Tinggi (Bagus)";
  if (tmp == P5) stat = "Sedang";
  if (tmp == Q5) stat = "Kurang";
  return {
    status: stat,
    tmp: tmp,
  };
}
</script>
<template>
  <div class="max-w-[1200px] p-4 pt-10 mx-auto">
    <div class="mb-10">
      <Label> Masukkan File Excel </Label>
      <Input type="file" @change="handleFileChange" />
    </div>
    <div v-if="dataLoaded">
      <Table>
        <TableHeader>
          <TableRow>
            <th>No.</th>
            <th>Tahun</th>
            <th>Kecamatan</th>
            <th>Nama</th>
            <th>Alokasi</th>
            <th>Realisasi</th>
            <th>Sisa/Kurang</th>
            <th>C1</th>
            <th>C2</th>
            <th>C3</th>
            <th>Kedekatan</th>
            <th>Hasil</th>
          </TableRow>
        </TableHeader>
        <TableBody>
          <TableRow v-for="(point, index) in points" :key="index">
            <TableCell>{{ index + 1 }}</TableCell>
            <TableCell>{{ point.tahun }}</TableCell>
            <TableCell>{{ point.kecamatan }}</TableCell>
            <TableCell>{{ point.nama }}</TableCell>

            <TableCell>{{ point.x }}</TableCell>
            <TableCell>{{ point.y }}</TableCell>
            <TableCell>{{ point.z }}</TableCell>
            <TableCell>{{ c1(point) }}</TableCell>
            <TableCell>{{ c2(point) }}</TableCell>
            <TableCell>{{ c3(point) }}</TableCell>
            <TableCell>{{
              kedekatan(c1(point), c2(point), c3(point))?.tmp
            }}</TableCell>
            <TableCell>{{
              kedekatan(c1(point), c2(point), c3(point))?.status
            }}</TableCell>
          </TableRow>
        </TableBody>
      </Table>
    </div>
  </div>
</template>

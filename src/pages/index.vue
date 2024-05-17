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
import { Button } from "@/components/ui/button";

const dataLoaded = ref(false);
const menu = ref(1);
const points = ref([]);
const knearest = ref([]);
const tester = ref([]);

const handleFileChange = (event) => {
  points.value = [];
  knearest.value = [];
  tester.value = [];
  const file = event.target.files[0];
  const reader = new FileReader();
  reader.onload = (e) => {
    const data = new Uint8Array(e.target.result);
    const workbook = XLSX.read(data, { type: "array" });
    const sheetName = workbook.SheetNames[0];
    const worksheet = workbook.Sheets[sheetName];
    const parsedData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
    const pointsData = parsedData.slice(1).map((row) => ({
      kode: row[1],
      tahun: row[2],
      f: row[3],
      g: row[4],
      h: row[5],
      i: row[6],
      j: row[7],
      k: row[8],
    }));
    console.log(parsedData);
    points.value = [...pointsData];
    const eightyPercentCount = Math.floor(pointsData.length * 0.8);
    const twentyPercentCount = Math.ceil(pointsData.length * 0.2);
    const tmp1 = [...pointsData];
    const tmp2 = [...pointsData];
    tester.value = [...tmp1.slice(-twentyPercentCount)];
    knearest.value = [...tmp2.slice(0, eightyPercentCount)];
    dataLoaded.value = true;
  };
  reader.readAsArrayBuffer(file);
};

const c1 = (point) => {
  return point.j / point.k;
};

const c2 = (point) => {
  return point.h / point.k;
};

const roa = (point) => {
  return point.i / point.k;
};

const gscore = (point) => {
  return 1.65 * c1(point) + 3.404 * c2(point) - 0.016 * roa(point) + 0.057;
};

function hasil(point) {
  return gscore(point) <= -0.02 ? "Bangkrut" : "Aman";
}

function jarak(point) {
  let tmp = tester.value[tester.value.length - 1];
  const diffB = c1(point) - c1(tmp);
  const diffC = c2(point) - c2(tmp);
  const diffD = roa(point) - roa(tmp);
  const diffE = gscore(point) - gscore(tmp);

  const distance = Math.sqrt(
    diffB * diffB + diffC * diffC + diffD * diffD + diffE * diffE
  );
  return distance;
}

function rank(value, arr, order = 1) {
  const sortedArr = [...arr].sort((a, b) => (order === 1 ? a - b : b - a));

  const ranks = new Map();
  for (let i = 0; i < sortedArr.length; i++) {
    const currentValue = sortedArr[i];
    if (!ranks.has(currentValue)) {
      const positions = sortedArr.reduce((acc, val, index) => {
        if (val === currentValue) {
          acc.push(index + 1);
        }
        return acc;
      }, []);
      const avgRank =
        positions.reduce((sum, pos) => sum + pos, 0) / positions.length;
      ranks.set(currentValue, avgRank);
    }
  }

  // Return the rank of the given value
  return ranks.get(value);
}
</script>
<template>
  <div class="max-w-[1200px] p-4 pt-10 mx-auto">
    <div class="mb-10">
      <Label> Masukkan File Excel </Label>
      <Input type="file" @change="handleFileChange" />
    </div>
    <div v-if="dataLoaded">
      <div class="flex space-x-4 mb-6">
        <!-- {{ points.length * 0.2 }} -->
        <Button @click="menu = 0" :variant="menu ? 'ghost' : ''"
          >G-Score</Button
        >
        <Button @click="menu = 1" :variant="!menu ? 'ghost' : ''"
          >K-Nearest</Button
        >
      </div>

      <Table :class="menu ? 'hidden' : ''">
        <TableHeader>
          <TableRow>
            <th>No.</th>
            <th>Kode</th>
            <th>Tahun</th>
            <th>Aset Lancar</th>
            <th>Utang Lancar</th>
            <th>Laba Kotor</th>
            <th>Laba Bersih</th>
            <th>Working Capita</th>
            <th>Total Aset</th>
            <th>X1</th>
            <th>X2</th>
            <th>ROA</th>
            <th>G-Score</th>
            <th>Hasil</th>
          </TableRow>
        </TableHeader>
        <TableBody>
          <TableRow v-for="(point, index) in points" :key="index">
            <TableCell>{{ index + 1 }}</TableCell>
            <TableCell>{{ point.kode }}</TableCell>
            <TableCell>{{ point.tahun }}</TableCell>

            <TableCell>{{ point.f }}</TableCell>
            <TableCell>{{ point.g }}</TableCell>
            <TableCell>{{ point.h }}</TableCell>

            <TableCell>{{ point.i }}</TableCell>
            <TableCell>{{ point.j }}</TableCell>
            <TableCell>{{ point.k }}</TableCell>

            <TableCell>{{ c1(point) }}</TableCell>
            <TableCell>{{ c2(point) }}</TableCell>
            <TableCell>{{ roa(point) }}</TableCell>
            <TableCell>{{ gscore(point) }}</TableCell>
            <TableCell
              ><span
                :class="
                  hasil(point) == 'Bangkrut'
                    ? 'text-rose-600 font-semibold'
                    : ''
                "
                >{{ hasil(point) }}</span
              ></TableCell
            >
          </TableRow>
        </TableBody>
      </Table>

      <Table :class="!menu ? 'hidden' : ''">
        <TableHeader>
          <TableRow>
            <th>No.</th>
            <th>X1</th>
            <th>X2</th>
            <th>ROA</th>
            <th>G-Score</th>
            <th>Hasil</th>

            <th>Jarak</th>
            <th>Rank</th>
          </TableRow>
        </TableHeader>
        <TableBody>
          <TableRow
            v-for="(point, index) in knearest"
            :key="index"
            :class="
              rank(
                jarak(point),
                knearest.map((e) => jarak(e)),
                1
              ) <= 5
                ? 'bg-amber-700/20'
                : ''
            "
          >
            <TableCell>{{ index + 1 }}</TableCell>
            <TableCell>{{ c1(point) }}</TableCell>
            <TableCell>{{ c2(point) }}</TableCell>
            <TableCell>{{ roa(point) }}</TableCell>
            <TableCell>{{ gscore(point) }}</TableCell>
            <TableCell
              ><span
                :class="
                  hasil(point) == 'Bangkrut'
                    ? 'text-rose-600 font-semibold'
                    : ''
                "
                >{{ hasil(point) }}</span
              ></TableCell
            >
            <TableCell>{{ jarak(point) }}</TableCell>
            <TableCell>{{
              rank(
                jarak(point),
                knearest.map((e) => jarak(e)),
                1
              )
            }}</TableCell>
          </TableRow>
        </TableBody>
      </Table>
    </div>
  </div>
</template>

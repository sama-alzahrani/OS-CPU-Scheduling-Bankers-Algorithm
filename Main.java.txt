import java.util.*;

public class Main {

    static class Job {
        int id;
        int arrival;
        int burst;
        int priority;
        int waiting;
        int turnaround;
        int completion;
        int remaining;

        Job(int id, int arrival, int burst, int priority) {
            this.id = id;
            this.arrival = arrival;
            this.burst = burst;
            this.priority = priority;
            this.remaining = burst;
        }

        Job copy() {
            return new Job(id, arrival, burst, priority);
        }
    }

    static class Result {
        List<Job> jobs;
        double avgWaiting;
        double avgTurnaround;
        String gantt;

        Result(List<Job> jobs, double aw, double at, String g) {
            this.jobs = jobs;
            this.avgWaiting = aw;
            this.avgTurnaround = at;
            this.gantt = g;
        }
    }

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);

        System.out.println("Operating Systems Project - CPU Scheduling and Banker's Algorithm");
        System.out.println("Student: Sama Hassan (44400233)");
        System.out.println();

        System.out.print("Enter number of processes: ");
        int n = in.nextInt();
        List<Job> base = new ArrayList<>();

        for (int i = 0; i < n; i++) {
            int id = i + 1;
            System.out.println("Process P" + id + ":");
            System.out.print("Arrival time: ");
            int at = in.nextInt();
            System.out.print("Burst time: ");
            int bt = in.nextInt();
            System.out.print("Priority (smaller = higher priority): ");
            int pr = in.nextInt();
            base.add(new Job(id, at, bt, pr));
            System.out.println();
        }

        System.out.print("Enter time quantum for Round Robin: ");
        int quantum = in.nextInt();
        System.out.println();

        Result rFcfs = fcfs(copyList(base));
        Result rSjf = sjf(copyList(base));
        Result rPrioNP = priorityNonPreemptive(copyList(base));
        Result rPrioP = priorityPreemptive(copyList(base));
        Result rRR = roundRobin(copyList(base), quantum);

        printResult("FCFS", rFcfs);
        printResult("SJF (Non-Preemptive)", rSjf);
        printResult("Priority Scheduling (Non-Preemptive)", rPrioNP);
        printResult("Priority Scheduling (Preemptive)", rPrioP);
        printResult("Round Robin (q = " + quantum + ")", rRR);

        drawCharts(rFcfs, rSjf, rPrioNP, rPrioP, rRR);

        bankersAlgorithm(in);
    }

    static List<Job> copyList(List<Job> src) {
        List<Job> out = new ArrayList<>();
        for (Job j : src) out.add(j.copy());
        return out;
    }

    static void computeWTandTAT(List<Job> jobs) {
        for (Job j : jobs) {
            j.turnaround = j.completion - j.arrival;
            j.waiting = j.turnaround - j.burst;
        }
    }

    static Result fcfs(List<Job> jobs) {
        jobs.sort(Comparator.comparingInt((Job j) -> j.arrival).thenComparingInt(j -> j.id));
        int time = 0;
        StringBuilder g = new StringBuilder();
        g.append("Gantt: ");

        for (int i = 0; i < jobs.size(); i++) {
            Job j = jobs.get(i);
            if (time < j.arrival) time = j.arrival;
            int start = time;
            time += j.burst;
            j.completion = time;
            g.append("P").append(j.id).append("[").append(start).append("->").append(time).append("]");
            if (i != jobs.size() - 1) g.append(" -> ");
        }

        computeWTandTAT(jobs);
        double[] avgs = averages(jobs);
        return new Result(jobs, avgs[0], avgs[1], g.toString());
    }

    static Result sjf(List<Job> jobs) {
        int time = 0;
        int done = 0;
        int n = jobs.size();
        StringBuilder g = new StringBuilder();
        g.append("Gantt: ");
        boolean firstSegment = true;

        int minArrival = Integer.MAX_VALUE;
        for (Job j : jobs) if (j.arrival < minArrival) minArrival = j.arrival;
        time = minArrival;

        boolean[] finished = new boolean[n];

        while (done < n) {
            Job pick = null;
            int idx = -1;
            for (int i = 0; i < n; i++) {
                Job j = jobs.get(i);
                if (!finished[i] && j.arrival <= time) {
                    if (pick == null || j.burst < pick.burst ||
                            (j.burst == pick.burst && j.arrival < pick.arrival)) {
                        pick = j;
                        idx = i;
                    }
                }
            }
            if (pick == null) {
                time++;
                continue;
            }
            int start = time;
            time += pick.burst;
            pick.completion = time;
            finished[idx] = true;
            done++;

            if (!firstSegment) g.append(" -> ");
            g.append("P").append(pick.id).append("[").append(start).append("->").append(time).append("]");
            firstSegment = false;
        }

        computeWTandTAT(jobs);
        double[] avgs = averages(jobs);
        return new Result(jobs, avgs[0], avgs[1], g.toString());
    }

    static Result priorityNonPreemptive(List<Job> jobs) {
        int time = 0;
        int done = 0;
        int n = jobs.size();
        StringBuilder g = new StringBuilder();
        g.append("Gantt: ");
        boolean firstSegment = true;

        int minArrival = Integer.MAX_VALUE;
        for (Job j : jobs) if (j.arrival < minArrival) minArrival = j.arrival;
        time = minArrival;

        boolean[] finished = new boolean[n];

        while (done < n) {
            Job pick = null;
            int idx = -1;
            for (int i = 0; i < n; i++) {
                Job j = jobs.get(i);
                if (!finished[i] && j.arrival <= time) {
                    if (pick == null || j.priority < pick.priority ||
                            (j.priority == pick.priority && j.arrival < pick.arrival)) {
                        pick = j;
                        idx = i;
                    }
                }
            }
            if (pick == null) {
                time++;
                continue;
            }
            int start = time;
            time += pick.burst;
            pick.completion = time;
            finished[idx] = true;
            done++;

            if (!firstSegment) g.append(" -> ");
            g.append("P").append(pick.id).append("[").append(start).append("->").append(time).append("]");
            firstSegment = false;
        }

        computeWTandTAT(jobs);
        double[] avgs = averages(jobs);
        return new Result(jobs, avgs[0], avgs[1], g.toString());
    }

    static Result priorityPreemptive(List<Job> jobs) {
        int n = jobs.size();
        int time = 0;
        int completed = 0;
        StringBuilder g = new StringBuilder();
        g.append("Gantt: ");

        int minArrival = Integer.MAX_VALUE;
        for (Job j : jobs) if (j.arrival < minArrival) minArrival = j.arrival;
        time = minArrival;

        for (Job j : jobs) j.remaining = j.burst;

        Job current = null;
        int segmentStart = time;
        boolean firstSegment = true;

        while (completed < n) {
            Job pick = null;
            for (Job j : jobs) {
                if (j.arrival <= time && j.remaining > 0) {
                    if (pick == null || j.priority < pick.priority ||
                            (j.priority == pick.priority && j.arrival < pick.arrival)) {
                        pick = j;
                    }
                }
            }

            if (pick == null) {
                time++;
                continue;
            }

            if (current == null || current.id != pick.id) {
                if (current != null) {
                    if (!firstSegment) g.append(" -> ");
                    g.append("P").append(current.id)
                            .append("[").append(segmentStart).append("->").append(time).append("]");
                    firstSegment = false;
                }
                current = pick;
                segmentStart = time;
            }

            pick.remaining--;
            time++;

            if (pick.remaining == 0) {
                pick.completion = time;
                completed++;
                if (current != null && current.id == pick.id) {
                    if (!firstSegment) g.append(" -> ");
                    g.append("P").append(current.id)
                            .append("[").append(segmentStart).append("->").append(time).append("]");
                    firstSegment = false;
                    current = null;
                }
            }
        }

        computeWTandTAT(jobs);
        double[] avgs = averages(jobs);
        return new Result(jobs, avgs[0], avgs[1], g.toString());
    }

    static Result roundRobin(List<Job> jobs, int quantum) {
        int n = jobs.size();
        for (Job j : jobs) j.remaining = j.burst;

        int time = 0;
        int completed = 0;
        StringBuilder g = new StringBuilder();
        g.append("Gantt: ");
        boolean firstSegment = true;

        int minArrival = Integer.MAX_VALUE;
        for (Job j : jobs) if (j.arrival < minArrival) minArrival = j.arrival;
        time = minArrival;

        Queue<Job> q = new ArrayDeque<>();
        boolean[] inQueue = new boolean[n];

        while (completed < n) {
            for (int i = 0; i < n; i++) {
                Job j = jobs.get(i);
                if (!inQueue[i] && j.arrival <= time && j.remaining > 0) {
                    q.add(j);
                    inQueue[i] = true;
                }
            }

            if (q.isEmpty()) {
                time++;
                continue;
            }

            Job cur = q.poll();
            int idx = cur.id - 1;
            int exec = Math.min(quantum, cur.remaining);
            int start = time;
            time += exec;
            cur.remaining -= exec;

            if (!firstSegment) g.append(" -> ");
            g.append("P").append(cur.id).append("[").append(start).append("->").append(time).append("]");
            firstSegment = false;

            for (int i = 0; i < n; i++) {
                Job j = jobs.get(i);
                if (!inQueue[i] && j.arrival <= time && j.remaining > 0) {
                    q.add(j);
                    inQueue[i] = true;
                }
            }

            if (cur.remaining > 0) {
                q.add(cur);
            } else {
                cur.completion = time;
                completed++;
            }
        }

        computeWTandTAT(jobs);
        double[] avgs = averages(jobs);
        return new Result(jobs, avgs[0], avgs[1], g.toString());
    }

    static double[] averages(List<Job> jobs) {
        double sumW = 0;
        double sumT = 0;
        for (Job j : jobs) {
            sumW += j.waiting;
            sumT += j.turnaround;
        }
        return new double[]{sumW / jobs.size(), sumT / jobs.size()};
    }

    static void printResult(String title, Result r) {
        System.out.println("=== " + title + " ===");
        System.out.println("PID | WT | TAT");
        for (Job j : r.jobs) {
            System.out.printf("P%-2d %3d %4d%n", j.id, j.waiting, j.turnaround);
        }
        System.out.printf("Average WT  = %.2f%n", r.avgWaiting);
        System.out.printf("Average TAT = %.2f%n", r.avgTurnaround);
        System.out.println(r.gantt);
        System.out.println();
    }

    static void drawCharts(Result fcfs, Result sjf, Result prioNP, Result prioP, Result rr) {
        System.out.println("=== Average Waiting Time Gantt-like Chart ===");
        drawBar("FCFS  ", fcfs.avgWaiting);
        drawBar("SJF   ", sjf.avgWaiting);
        drawBar("PrioNP", prioNP.avgWaiting);
        drawBar("PrioP ", prioP.avgWaiting);
        drawBar("RR    ", rr.avgWaiting);
        System.out.println();

        System.out.println("=== Average Turnaround Time Gantt-like Chart ===");
        drawBar("FCFS  ", fcfs.avgTurnaround);
        drawBar("SJF   ", sjf.avgTurnaround);
        drawBar("PrioNP", prioNP.avgTurnaround);
        drawBar("PrioP ", prioP.avgTurnaround);
        drawBar("RR    ", rr.avgTurnaround);
        System.out.println();
    }

    static void drawBar(String name, double value) {
        int len = (int) Math.round(value * 2);
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < len; i++) sb.append('#');
        System.out.printf("%s | %s (%.2f)%n", name, sb.toString(), value);
    }

    static void bankersAlgorithm(Scanner in) {
        System.out.println("=== Banker's Algorithm Simulation ===");
        System.out.print("Enter number of processes: ");
        int n = in.nextInt();
        System.out.print("Enter number of resource types: ");
        int m = in.nextInt();

        int[][] max = new int[n][m];
        int[][] alloc = new int[n][m];
        int[][] need = new int[n][m];
        int[] avail = new int[m];

        System.out.println("Enter Max matrix (" + n + "x" + m + "):");
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                max[i][j] = in.nextInt();

        System.out.println("Enter Allocation matrix (" + n + "x" + m + "):");
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                alloc[i][j] = in.nextInt();

        System.out.println("Enter Available vector (" + m + " values):");
        for (int j = 0; j < m; j++)
            avail[j] = in.nextInt();

        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                need[i][j] = max[i][j] - alloc[i][j];

        System.out.println("Need matrix:");
        for (int i = 0; i < n; i++) {
            System.out.print("P" + (i + 1) + ": ");
            for (int j = 0; j < m; j++)
                System.out.print(need[i][j] + " ");
            System.out.println();
        }

        boolean[] finished = new boolean[n];
        List<Integer> safeSeq = new ArrayList<>();
        int[] work = Arrays.copyOf(avail, m);
        boolean progress;

        while (safeSeq.size() < n) {
            progress = false;
            for (int i = 0; i < n; i++) {
                if (finished[i]) continue;
                boolean can = true;
                for (int j = 0; j < m; j++) {
                    if (need[i][j] > work[j]) {
                        can = false;
                        break;
                    }
                }
                if (can) {
                    for (int j = 0; j < m; j++)
                        work[j] += alloc[i][j];
                    finished[i] = true;
                    safeSeq.add(i);
                    progress = true;
                }
            }
            if (!progress) break;
        }

        boolean safe = safeSeq.size() == n;
        if (!safe) {
            System.out.println("System state: UNSAFE (no safe sequence)");
        } else {
            System.out.println("System state: SAFE");
            System.out.print("Safe sequence: ");
            for (int i = 0; i < safeSeq.size(); i++) {
                System.out.print("P" + (safeSeq.get(i) + 1));
                if (i != safeSeq.size() - 1) System.out.print(" -> ");
            }
            System.out.println();
        }
    }
}

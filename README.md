#
# Live: https://sofax.vercel.app/
exam/[exam-id]/page.tsx -------import { notFound } from "next/navigation";
import ExamDetails from "../exam-details";

// Simulated API call or database query for fetching exam details
const getExamData = async (id: string) => {
  // Example exam data tailored for the updated component
  const exam = {
    id: parseInt(id),
    examName: "Final Semester Exam",
    examDate: "2024-12-15",
    durationInMinutes: 120,
    totalMarks: 100,
    passingMarks: 40,
    subject: "Mathematics",
    examRules: [
      "Arrive at the exam hall 30 minutes early.",
      "Carry only necessary stationery items.",
      "Electronic devices are not allowed.",
      "Follow invigilator instructions strictly.",
    ],
    additionalNotes: "Ensure you revise the entire syllabus before the exam.",
  };

  return exam;
};

export default async function ExamPage({
  params,
}: {
  params: { id: string };
}) {
  const exam = await getExamData(params.id);

  if (!exam) {
    notFound();
  }

  return <ExamDetails exam={exam} />;
}----------------------------------------------------------------------
exam/exam-details.tsx ------"use client";

import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import ExamMonitor from "./exam-monitor";

type ExamDetailsProps = {
    exam: {
        id: number;
        examName: string;
        examDate: string;
        durationInMinutes: number;
        totalMarks: number;
        passingMarks: number;
        subject: string;
        examRules: string[];
        additionalNotes?: string;
    };
};

export default function ExamDetails({ exam }: ExamDetailsProps) {
    return (
        <div>
            <div className="container mx-auto py-3 bg-gradient-to-br from-blue-50 to-indigo-200 rounded-b-xl shadow-lg">
                <div className="flex justify-between items-center mb-2">
                    <div>
                        <h1 className=" font-semibold text-gray-800">
                            LearningART | <span className="text-blue-500">Assessment Dashboard</span>
                        </h1>
                    </div>
                    <div className="flex space-x-6 text-sm text-gray-600">
                        <p><strong>Subject:</strong> {exam.subject}</p>
                        <p><strong>Exam Date:</strong> {exam.examDate}</p>
                        <p><strong>Duration:</strong> {exam.durationInMinutes} minutes</p>
                        <p><strong>Total Marks:</strong> {exam.totalMarks}</p>
                        <p><strong>Passing Marks:</strong> {exam.passingMarks}</p>
                    </div>
                </div>
                <div className="flex justify-between items-center">
                    <h3 className="text-xl font-semibold text-gray-800">
                        Exam Name: <span className="text-lg font-medium text-gray-900 ml-1">{exam.examName}</span>
                    </h3>
                    <h3 className="text-xl font-semibold text-gray-800">
                        Student Name: <span className="text-lg font-medium text-gray-900 ml-1">{"Asif"}</span>
                    </h3>
                </div>
            </div>

            <ExamMonitor totalQuestions={50}></ExamMonitor>
        </div>
    );
}--------------------------------------------------------------
exam/exam-monitor.tsx---------import React, { useState, useEffect } from "react";

interface ExamMonitorProps {
  totalQuestions: number;
}

const ExamMonitor: React.FC<ExamMonitorProps> = ({ totalQuestions }) => {
  const [isExamActive, setIsExamActive] = useState<boolean>(false);
  const [timeRemaining, setTimeRemaining] = useState<number>(3600); // 1 hour in seconds
  const [answeredQuestions, setAnsweredQuestions] = useState<number>(0);
  const [questionTimers, setQuestionTimers] = useState<number[]>(new Array(totalQuestions).fill(0));
  const [questionActive, setQuestionActive] = useState<number | null>(null);

  // Start and stop exam
  const handleStartExam = (): void => {
    setIsExamActive(true);
  };

  const handleStopExam = (): void => {
    setIsExamActive(false);
  };

  // Timer countdown effect
  useEffect(() => {
    let interval: NodeJS.Timeout | undefined;

    if (isExamActive && timeRemaining > 0) {
      interval = setInterval(() => {
        setTimeRemaining((prevTime) => prevTime - 1);
      }, 1000);
    } else if (!isExamActive) {
      if (interval) clearInterval(interval);
    }

    return () => {
      if (interval) clearInterval(interval); // Cleanup on component unmount
    };
  }, [isExamActive, timeRemaining]);

  // Handle question click (start timer for that question)
  const handleQuestionClick = (index: number): void => {
    if (!isExamActive || questionActive === index) return; // Prevent answering if the exam is stopped or timer for the question is active

    setQuestionActive(index);
    setAnsweredQuestions((prev) => prev + 1); // Increase the count of answered questions
  };

  // Update time for active question
  useEffect(() => {
    let interval: NodeJS.Timeout | undefined;

    if (questionActive !== null) {
      interval = setInterval(() => {
        const updatedTimers = [...questionTimers];
        updatedTimers[questionActive] += 1; // Increment time for the active question
        setQuestionTimers(updatedTimers);
      }, 1000);
    }

    return () => {
      if (interval) clearInterval(interval); // Cleanup on unmount or question change
    };
  }, [questionActive, questionTimers]);

  // Format time in HH:MM:SS format
  const formatTime = (seconds: number): string => {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const remainingSeconds = (seconds % 60).toFixed(2);
    return `${String(hours).padStart(2, "0")}:${String(minutes).padStart(2, "0")}:${remainingSeconds.padStart(5, "0")}`;
  };

  return (
    <div className="py-3 w-full">
      <div className="grid gap-4 grid-cols-4">
        {/* Time Start/Stop */}
        <div className="bg-blue-100 p-2 rounded-lg shadow-md flex flex-col items-center">
          <h2 className="text-lg font-semibold mb-2">Start/Stop Exam</h2>
          {!isExamActive ? (
            <button
              className="bg-blue-500 text-white px-4 py-2 rounded-lg shadow hover:bg-blue-600 h-10"
              onClick={handleStartExam}
            >
              Start Exam
            </button>
          ) : (
            <button
              className="bg-red-500 text-white px-4 py-2 rounded-lg shadow mt-3 hover:bg-red-600 h-10"
              onClick={handleStopExam}
            >
              Stop Exam
            </button>
          )}
        </div>

        {/* Timer */}
        <div className="bg-green-100 p-2 rounded-lg shadow-md flex flex-col items-center">
          <h2 className="text-lg font-semibold mb-2">Time Remaining</h2>
          <div className="text-2xl font-bold text-gray-700">{formatTime(timeRemaining)}</div>
          <p className="mt-1 text-sm text-gray-600">Time Left</p>
        </div>

        {/* Answered Question Level */}
        <div className="bg-yellow-100 p-2 rounded-lg shadow-md flex flex-col items-center">
          <h2 className="text-lg font-semibold mb-2">Answered Questions</h2>
          <div className="text-2xl font-bold text-gray-700">
            {answeredQuestions}/{totalQuestions}
          </div>
          <p className="mt-1 text-sm text-gray-600">Questions Answered</p>
        </div>

        {/* Average Time Per Question */}
        <div className="bg-indigo-100 p-2 rounded-lg shadow-md flex flex-col items-center">
          <h2 className="text-lg font-semibold mb-2">Average Time per Question</h2>
          <div className="text-2xl font-bold text-gray-700">
            {formatTime(
              Math.round(
                (questionTimers.reduce((acc, time) => acc + time, 0) / totalQuestions) * 100
              ) / 100
            )}
          </div>
          <p className="mt-1 text-sm text-gray-600">Average Time per Answer</p>
        </div>
      </div>

      <div className="mt-4">
        <h2 className="text-lg font-semibold mb-4">Questions</h2>
        <div className="grid grid-cols-2 gap-4">
          {Array.from({ length: totalQuestions }, (_, index) => (
            <div
              key={index}
              className={`p-4 rounded-lg shadow-md cursor-pointer ${isExamActive ? 'hover:bg-green-200' : 'bg-gray-300'} ${questionActive === index ? "bg-green-200" : "bg-gray-200"}`}
              onClick={() => handleQuestionClick(index)}
              style={{ pointerEvents: isExamActive ? "auto" : "none" }}  // Disable clicking when exam is stopped
            >
              <h3 className="text-lg font-semibold">Question {index + 1}</h3>
              <p className="text-sm text-gray-600">Time: {formatTime(questionTimers[index])}</p>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};

export default ExamMonitor;---------------------------------------------------------
exam-list/page.tsx--------"use client";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";
import { ArrowUpDown, Play } from "lucide-react";
import Link from "next/link";
import { useState } from "react";

const mockData = [
  {
    id: 1,
    exam_id: 101,
    exam_name: "Mathematics",
    exam_type: "true-false",
    student_id: 5001,
    assigned_by_tutor_id: 3001,
    tutor_name: "John",
    institute_name: "Dhaka University",
    institute_id: 2001,
    status_completed: true,
    schedule_exam_datetime: "2024-11-01T10:00:00",
    actual_exam_datetime: "2024-11-01T10:30:00",
    student_exam_duration: 30,
    student_exam_score: 85,
    student_number_of_questions_answered: 25,
    student_number_of_correct_answers: 20,
    student_exam_report: "Great performance",
    exam_remaining: true,
  },
  {
    id: 2,
    exam_id: 102,
    exam_name: "English",
    student_id: 5002,
    exam_type: "MCQ",
    assigned_by_tutor_id: 3002,
    tutor_name: "Asif",
    institute_id: 2002,
    institute_name: "Jahangirnagar University",
    status_completed: false,
    schedule_exam_datetime: "2024-11-02T11:00:00",
    actual_exam_datetime: null,
    student_exam_duration: 40,
    student_exam_score: null,
    student_number_of_questions_answered: null,
    student_number_of_correct_answers: null,
    student_exam_report: "Good",
    exam_remaining: true,
  },
  {
    id: 3,
    exam_id: 103,
    exam_name: "Chemistry",
    student_id: 5003,
    exam_type: "writing-answer",
    assigned_by_tutor_id: 3002,
    tutor_name: "Tarif",
    institute_id: 2002,
    institute_name: "Jagannath University",
    status_completed: false,
    schedule_exam_datetime: "2024-11-02T11:00:00",
    actual_exam_datetime: null,
    student_exam_duration: 40,
    student_exam_score: null,
    student_number_of_questions_answered: null,
    student_number_of_correct_answers: null,
    student_exam_report: null,
    exam_remaining: true,
  },
  {
    id: 4,
    exam_id: 104,
    exam_name: "Physics",
    student_id: 5004,
    exam_type: "match-images",
    assigned_by_tutor_id: 3002,
    tutor_name: "Taki",
    institute_id: 2002,
    institute_name: "Brack University",
    status_completed: false,
    schedule_exam_datetime: "2024-11-02T11:00:00",
    actual_exam_datetime: null,
    student_exam_duration: 40,
    student_exam_score: null,
    student_number_of_questions_answered: null,
    student_number_of_correct_answers: null,
    student_exam_report: "Mediocore",
    exam_remaining: true,
  },
];

export default function ExamStudentReport() {
  const [data, setData] = useState(mockData);
  const [search, setSearch] = useState("");
  const [sortConfig, setSortConfig] = useState<{
    key: string;
    direction: "asc" | "desc" | null;
  }>({
    key: "",
    direction: null,
  });

  const filteredData = data.filter(
    (item) =>
      item.exam_id.toString().includes(search) ||
      item.student_id.toString().includes(search)
  );

  const handleSort = (key: string) => {
    let direction: "asc" | "desc" | null = "asc";
    if (sortConfig.key === key && sortConfig.direction === "asc") {
      direction = "desc";
    } else if (sortConfig.key === key && sortConfig.direction === "desc") {
      direction = null;
    }

    setSortConfig({ key, direction });

    if (direction) {
      const sortedData = [...filteredData].sort((a, b) => {
        const aValue = a[key as keyof typeof a];
        const bValue = b[key as keyof typeof b];
        if (typeof aValue === "number" && typeof bValue === "number") {
          return direction === "asc" ? aValue - bValue : bValue - aValue;
        } else if (typeof aValue === "string" && typeof bValue === "string") {
          return direction === "asc"
            ? aValue.localeCompare(bValue)
            : bValue.localeCompare(aValue);
        }
        return 0;
      });
      setData(sortedData);
    } else {
      setData(mockData); // Reset to original data if sorting is removed
    }
  };

  return (
    <div className="max-w-[98%] mx-auto p-4">
      {/* Header */}
      <header className="bg-white shadow">
        <div className="container mx-auto px-6 py-4 flex justify-between items-center">
          {/* Left Section: Student Info */}
          <div className="flex items-center space-x-4">
            <img
              width={500}
              height={500}
              className="size-32 rounded-md bg-slate-500 object-cover"
              src="https://images.unsplash.com/photo-1633332755192-727a05c4013d?q=80&w=2080&auto=format&fit=crop"
              alt="student profile"
            />
            <div>
              <h3 className="text-xl font-semibold text-gray-800">John Doe</h3>
              <p className="font-semibold text-gray-600">Grade: 12th</p>
              <p className="font-semibold text-gray-600">Student ID: 123456</p>
              <p className="font-semibold text-gray-600">Zone: New York, United States</p>
              <div className="flex space-x-4 mt-1">
                <button className="bg-blue-500 text-white px-4 py-1 rounded-lg shadow hover:bg-blue-600 transition">
                  LearningART
                </button>
                <button className="bg-green-500 text-white px-4 py-1 rounded-lg shadow hover:bg-green-600 transition">
                  ScholerPASS
                </button>
                <button className="bg-yellow-500 text-white px-4 py-1 rounded-lg shadow hover:bg-yellow-600 transition">
                  Mathematics
                </button>
                <button className="bg-red-500 text-white px-4 py-1 rounded-lg shadow hover:bg-red-600 transition">
                  Chemistry
                </button>
                <button className="bg-purple-500 text-white px-4 py-1 rounded-lg shadow hover:bg-purple-600 transition">
                  Biology
                </button>
              </div>

            </div>
          </div>

          {/* Right Section: Dashboard Title */}
          <h2 className="text-xl text-gray-800 flex">
            <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6 text-green-500 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 10h4v10H3zM7 10V5a2 2 0 012-2h10a2 2 0 012 2v15a2 2 0 01-2 2H9a2 2 0 01-2-2V10z" />
            </svg>
            Online
          </h2>

        </div>
      </header>
      <div className="grid grid-cols-12 gap-0 w-full border-2 rounded-md border-gray-500 mb-5 mt-5">
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          Dashboard
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          Courses
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          Tutors
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          Tutoring Sessions
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          Assignments
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          Exams
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          Connect
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          ScholarPASS
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          LearningART
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          Referrals
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          Student Editor
        </button>
        <button className="flex items-center justify-center h-12 border border-gray-500 text-blue-500 font-semibold">
          Admin
        </button>
      </div>
      <div className="flex justify-between items-center mb-4">
        <h1 className="text-lg font-semibold">Exam Reports</h1>
        <Input
          placeholder="Search by Exam ID or Student ID..."
          value={search}
          onChange={(e) => setSearch(e.target.value)}
          className="w-56"
        />
      </div>

      {/* Table Wrapper */}
      <div className="overflow-auto">
        <Table className="min-w-[900px]">
          <TableHeader>
            <TableRow>
              <TableHead className="">
                <Button
                  variant="ghost"
                  size="sm"
                  onClick={() => handleSort("exam_name")}
                >
                  Exam <ArrowUpDown className="h-2 w-2  inline" />
                </Button>
              </TableHead>
              <TableHead className="">
                <Button
                  variant="ghost"
                  size="sm"
                  onClick={() => handleSort("schedule_exam_datetime")}
                >
                  Scheduled <ArrowUpDown className="h-2 w-2  inline" />
                </Button>
              </TableHead>
              <TableHead className="">
                <span className="text-xs">Exam Performance</span>
              </TableHead>
              <TableHead className="">
                <span className="text-xs">Test Exam</span>
              </TableHead>
              <TableHead className="">
                <span className=" text-xs">Status</span>
              </TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {filteredData.length ? (
              filteredData.map((item) => (
                <TableRow key={item.id}>
                  <TableCell className="w-[300px]"><span>{item.exam_name}</span><br /><span>Tutor: {item.tutor_name}</span><br /><span>Institue: {item.institute_name}</span>
                  </TableCell>
                  <TableCell>
                    <span>Date: {new Date(item.schedule_exam_datetime).toLocaleDateString()}</span> <br />
                    <span>Actual: {item.actual_exam_datetime
                      ? new Date(item.actual_exam_datetime).toLocaleDateString()
                      : "N/A"}</span> <br />
                    <span>Duration: {item.student_exam_duration || "N/A"}</span>
                  </TableCell>
                  <TableCell>
                    <span>Score: {item.student_exam_score || "N/A"}</span> <br />
                    <span>Questions: {item.student_number_of_questions_answered || "N/A"}</span> <br />
                    <span>Correct: {item.student_number_of_correct_answers || "N/A"}</span>
                  </TableCell>
                  <TableCell className="">
                    <p>
                      <span className="">{item.student_exam_report ? (
                        <Link
                          href={`exam-list/exam/${item.exam_id}`}
                        >
                          <button className="px-2 rounded-sm bg-blue-400 text-white">View</button>
                        </Link>
                      ) : (
                        "N/A"
                      )}</span>
                      <br />
                      <span className=" pb-[10px]">
                        {item.exam_remaining ? (
                          <Link
                            href={`exam-list/exam/${item.exam_id}/${item.exam_type}`}
                          >
                            <button className="px-2 rounded-sm bg-blue-400 text-white mt-2">Start</button>
                          </Link>
                        ) : (
                          <span>Already Taken </span>
                        )}
                      </span>
                    </p>
                  </TableCell>
                  <TableCell>
                    <Badge
                      variant={
                        item.status_completed ? "default" : "destructive"
                      }
                    >
                      {item.status_completed ? "Completed" : "Pending"}
                    </Badge>
                  </TableCell>
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell colSpan={12} className="text-center">
                  No reports found.
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </div>
    </div>
  );
}--------------------------------------------------------------------


// ----------------------------------------------------------------
// Copyright (c) 2003,成都天奥信息科技有限公司
// All rights reserved. 
//
// 文件名称：Log.cs
// 功能描述：日志类，写日志首先利用队列缓存日志，然后在线程中将日志写入文本文件
// 当前版本：v1.0
// 创建标识：何四海20151014
//
// 当前版本：V1.1
// 修改标识：漆帅20160304
// 修改描述：将成员变量将静态改为非静态，解决无法创建多个实例使用的问题
//
// 当前版本：V1.2
// 修改标识：刘宇20160519
// 修改描述：修改命名空间
//
// 修改版本：V1.3
// 修改标识：卢新平20160530
// 修改描述：增加了日志目录不存在就新生成一个Log目录的功能
//
// 修改版本：V1.4
// 修改标识：何四海20160826
// 修改描述：增加了记录日志委托定义
// ----------------------------------------------------------------
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;
using System.Threading;

namespace Spaceon.CommonUtils.LogRecord
{
    #region 委托定义区
    /// <summary>
    /// 日志委托
    /// </summary>
    /// <param name="logType">日志类型，0：未知，1：普通消息，2：警告，3：错误，4：成功，其他：未知</param>
    /// <param name="logContent">日志内容</param>
    public delegate void DegLogRecord(byte logType, string logContent);
    #endregion 委托定义区

    /// <summary>
    /// 日志类，提供日志记录实现
    /// </summary>
    public class Log : IDisposable
    {
        #region 成员变量

        // 类实例（单一类）
        private static Log _instance = null; 

        // 日志对象缓存队列
        private  Queue<Msg> _msgs;
        // 日志文件保存路径
        private  string _path;
        // 日志写入线程的控制标记
        private  bool _state;
        // 日志记录类型
        private  LogType _type;
        // 日志文件生命周期的时间标记
        private  DateTime _timeSign;
        // 日志文件写入流对象
        private  StreamWriter _writer;
        #endregion

        #region 构造函数
        
        /// <summary>
        /// 创建日志对象的新实例，采用默认当前程序位置作为日志路径和默认的每日日志记录类型记录日志
        /// </summary>
        public Log()
            :this(System.AppDomain.CurrentDomain.BaseDirectory + "\\Log\\", LogType.Daily)
        {
        }

        /// <summary>
        /// 创建日志对象的新实例，采用默认当前程序位置作为日志路径
        /// </summary>
        /// <param name="logType">日志记录类型</param>
        public Log(LogType logType)
            : this(System.AppDomain.CurrentDomain.BaseDirectory + "\\Log\\", logType)
        {
        }

        /// <summary>
        /// 创建日志对象的新实例，默认为1天写一个日志文件
        /// </summary>
        /// <param name="path">日志路径存放</param>
        public Log(string path)
            :this(path, LogType.Daily)
        {
        }

        /// <summary>
        /// 创建日志对象的新实例
        /// </summary>
        /// <param name="path">日志路径</param>
        /// <param name="logType">日志记录类型</param>
        public Log(string path, LogType logType)
        {
            _path = path;
            _type = logType;
            // 如果日志路径不存在，就创建一个Log路径,卢新平20160530
            if (!Directory.Exists(_path))
            {
                DirectoryInfo directoryInfo = new DirectoryInfo(_path);
                directoryInfo.Create();
            }
        }

        /// <summary>
        /// 获取类实例(单一类)
        /// </summary>
        /// <returns>日志实例</returns>
        public static Log GetInstance()
        {
            if (_instance == null)
            {
                _instance = new Log();
            }
            return _instance;
        }

        #endregion

        #region 析构函数

        /// <summary>
        /// 析构函数，终止写入线程并关闭文件
        /// </summary>
        ~Log()
        {
            if (_state == true)
            {
                _state = false;
            }
        }

        #endregion

        #region 公有函数

        /// <summary>
        /// 初始化
        /// </summary>
        public void StartWriteLog()
        {
            if (_msgs == null)
            {
                _state = true;
                _msgs = new Queue<Msg>();
                Thread thread = new Thread(Work);
                thread.Start();
            }
        }

        /// <summary>
        /// 写入新日志
        /// </summary>
        /// <param name="msg">日志对象</param>
        public void Write(Msg msg)
        {
            if (msg != null && _msgs != null)
            {
                lock (_msgs)
                {
                    _msgs.Enqueue(msg);
                }
            }
        }

        /// <summary>
        /// 写入新日志，采用当前时间为日志写入时间，日志类型默认为“消息”类型
        /// </summary>
        /// <param name="text">日志内容</param>
        public void Write(string text)
        {
            MsgType type = MsgType.Information;
            Write(new Msg(text, type));
        }

        /// <summary>
        /// 写入新日志，根据日志内容和信息类型，采用当前时间为日志写入时间
        /// </summary>
        /// <param name="text">日志内容</param>
        /// <param name="type">日志类型</param>
        public void Write(string text, MsgType type)
        {
            Write(new Msg(text, type));
        }

        /// <summary>
        /// 写入新日志，根据指定的日志时间、日志内容和信息类型写入新日志
        /// </summary>
        /// <param name="datetime">日志时间</param>
        /// <param name="text">日志内容</param>
        /// <param name="type">信息类型</param>
        public void Write(DateTime datetime, string text, MsgType type)
        {
            Write(new Msg(datetime, text, type));
        }
        #endregion

        #region 私有函数

        /// <summary>
        /// 日志写入线程执行方法
        /// </summary>
        private void Work()
        {
            while (true)
            {
                // 判断队列中是否存在待写入的日志
                if (_msgs.Count > 0)
                {
                    Msg msg = null;
                    lock (_msgs)
                    {
                        msg = _msgs.Dequeue();
                    }
                    if (msg != null)
                    {
                        FileWrite(msg);
                    }
                }
                else
                {
                    Thread.Sleep(100);
                }

                // 判断是否已经发出终止日志并关闭的消息
                if (!_state)
                {
                    int msgCount = _msgs.Count;
                    for (int i = 0; i < msgCount; i++)
                    {
                        Msg msg = null;
                        lock (_msgs)
                        {
                            msg = _msgs.Dequeue();
                        }
                        if (msg != null)
                        {
                            FileWrite(msg);
                        }
                    }
                    FileClose();
                    break;
                }
            }
        }

        /// <summary>
        /// 根据日志记录类型获取日志文件名，并同时创建文件到期时间标志
        /// 通过判断文件的到期时间标志决定是否创建新文件
        /// </summary>
        /// <returns>日志文件名</returns>
        private string GetFileName()
        {
            DateTime now = DateTime.Now;
            string format = "";
            switch (_type)
            {
                case LogType.Daily:
                    _timeSign = new DateTime(now.Year, now.Month, now.Day);
                    _timeSign.AddDays(1);
                    format = "yyyyMMdd";
                    break;
                case LogType.Weekly:
                    _timeSign = new DateTime(now.Year, now.Month, now.Day);
                    _timeSign.AddDays(7);
                    format = "yyyyMMdd";
                    break;
                case LogType.Monthly:
                    _timeSign = new DateTime(now.Year, now.Month, 1);
                    _timeSign.AddMonths(1);
                    format = "yyyyMM";
                    break;
                case LogType.Annually:
                    _timeSign = new DateTime(now.Year, 1, 1);
                    _timeSign.AddYears(1);
                    format = "yyyy";
                    break;
            }
            return now.ToString(format) + ".log";
        }

        /// <summary>
        /// 写入日志到文件中
        /// </summary>
        /// <param name="msg">消息</param>
        private void FileWrite(Msg msg)
        {
            try
            {
                if (_writer == null)
                {
                    FileOpen();
                    _writer.Write(msg.ToString());
                    _writer.Flush();
                }
                else
                {
                    // 判断日志到期标志，如果当前文件到期则关闭当前文件创建新的日志文件
                    if (DateTime.Now >= _timeSign)
                    {
                        FileClose();
                        FileOpen();
                    }

                    _writer.Write(msg.ToString());
                    _writer.Flush();
                }
            }
            catch (System.Exception ex)
            {
                Console.Out.Write(ex);
            }
        }

        /// <summary>
        /// 打开文件准备写入
        /// </summary>
        private void FileOpen()
        {
            _writer = new StreamWriter(_path + GetFileName(), true, Encoding.UTF8);
        }

        /// <summary>
        /// 关闭打开的日志文件
        /// </summary>
        private void FileClose()
        {
            if (_writer != null)
            {
                _writer.Flush();
                _writer.Close();
                _writer.Dispose();
                _writer = null;
            }
        }

        #endregion

        #region IDisposable成员

        /// <summary>
        /// 调用后终止线程并关闭日志文件
        /// </summary>
        public void Dispose()
        {
            _state = false;
        }
        #endregion

    }

    /// <summary>
    /// 表示一个日志记录对象
    /// </summary>
    public class Msg
    {
        #region 成员变量
        // 日志记录的时间
        private DateTime _datetime;
        // 日志记录的内容
        private string _text;
        // 日志记录的类型
        private MsgType _type;
        #endregion


        #region 构造函数
        /// <summary>
        /// 创建新的日志记录实例，日志记录内容为空，消息类型为MsgType.Unknown，日志时间为当前时间
        /// </summary>
        public Msg()
            :this("", MsgType.Unknown)
        {
        }

        /// <summary>
        /// 创建新的日志记录实例，日志时间为当前时间
        /// </summary>
        /// <param name="text">日志记录的文本内容</param>
        /// <param name="msgType">日志记录的消息类型</param>
        public Msg(string text, MsgType msgType)
            :this(DateTime.Now, text, msgType)
        {
        }

        /// <summary>
        /// 创建新的日志记录实例
        /// </summary>
        /// <param name="datetime">日志记录的时间</param>
        /// <param name="text">日志记录的文本内容</param>
        /// <param name="msgType">日志记录的消息类型</param>
        public Msg(DateTime datetime, string text, MsgType msgType)
        {
            _datetime = datetime;
            _text = text;
            _type = msgType;
        }
        #endregion

        #region 属性
        /// <summary>
        /// 获取或设置日志记录的时间
        /// </summary>
        public DateTime DateTime
        {
            get { return _datetime; }
            set { _datetime = value; }
        }

        /// <summary>
        /// 获取或设置日志记录的文本内容
        /// </summary>
        public string Text
        {
            get { return _text; }
            set { _text = value; }
        }

        /// <summary>
        /// 获取或设置日志类型
        /// </summary>
        public MsgType Type
        {
            get { return _type; }
            set { _type = value; }
        }
        #endregion

        #region 公有方法
        /// <summary>
        /// 获取消息字符串
        /// </summary>
        /// <returns>消息字符串</returns>
        public new string ToString()
        {
            string type = "";
            if (_type == MsgType.Error)
            {
                type = "错误";
            }
            else if (_type == MsgType.Warning)
            {
                type = "警告";
            }
            else if (_type == MsgType.Information)
            {
                type = "消息";
            }
            else if (_type == MsgType.Success)
            {
                type = "成功";
            }
            else
            {
                type = "未知";
            }
            return _datetime.ToString("yyyy-MM-dd HH:mm:ss") + "\t" + type + "\t" + _text + "\r\n";
        }
        #endregion
    }


    /// <summary>
    /// 日志文件创建类型枚举
    /// </summary>
    /// <remarks>
    /// 日志枚举类型指示日志文件创建的方式，
    /// 如果日志比较多可考虑每天创建一个日志文件，
    /// 如果日志量比较小可考虑每周、每月或每年创建一个日志文件。
    /// </remarks>
    public enum LogType
    {
        /// <summary>
        /// 指示每天创建一个新的日志文件
        /// </summary>
        Daily,

        /// <summary>
        /// 指示每周创建一个新的日志文件
        /// </summary>
        Weekly,

        /// <summary>
        /// 指示每月创建一个新的日志文件
        /// </summary>
        Monthly,

        /// <summary>
        /// 指示每年创建一个新的日志文件
        /// </summary>
        Annually
    }


    /// <summary>
    /// 日志消息类型的枚举
    /// </summary>
    public enum MsgType
    {
        /// <summary>
        /// 指示未知消息类型的日志记录
        /// </summary>
        Unknown,

        /// <summary>
        /// 指示普通消息类型的日志记录
        /// </summary>
        Information,

        /// <summary>
        /// 指示警告信息类型的日志记录
        /// </summary>
        Warning,

        /// <summary>
        /// 指示错误信息类型的日志记录
        /// </summary>
        Error,

        /// <summary>
        /// 指示成功信息类型的日志记录
        /// </summary>
        Success
    }
}

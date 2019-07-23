namespace MyTested.AspNetCore.Mvc
{
    using System.IO;
    using System.Linq;
    using Builders.ActionResults.File;
    using Builders.Base;
    using Builders.Contracts.ActionResults.File;
    using Exceptions;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.FileProviders;
    using Utilities;
    using Utilities.Extensions;

    /// <summary>
    /// Contains extension methods for <see cref="IFileTestBuilder"/>.
    /// </summary>
    public static class FileTestBuilderExtensions
    {
        private const string FileStream = "stream";
        private const string FileName = "name";
        private const string FileProvider = "provider";
        private const string FileContents = "contents";

        /// <summary>
        /// Tests whether the <see cref="FileStreamResult"/>
        /// has the same file stream as the provided one.
        /// </summary>
        /// <param name="fileTestBuilder">
        /// Instance of <see cref="IFileTestBuilder"/> type.
        /// </param>
        /// <param name="stream">File stream.</param>
        /// <returns>The same <see cref="IAndFileTestBuilder"/>.</returns>
        public static IAndFileTestBuilder WithStream(
            this IFileTestBuilder fileTestBuilder,
            Stream stream)
        {
            var actualBuilder = GetFileTestBuilder<FileStreamResult>(fileTestBuilder, FileStream);

            var fileStreamResult = actualBuilder.ActionResult;
            var expectedContents = stream.ToByteArray();
            var actualContents = fileStreamResult.FileStream.ToByteArray();

            if (!expectedContents.SequenceEqual(actualContents))
            {
                actualBuilder.ThrowNewFailedValidationException(
                    FileStream,
                    "to have value as the provided one",
                    "instead received different result");
            }

            return actualBuilder;
        }

        /// <summary>
        /// Tests whether the <see cref="VirtualFileResult"/>
        /// has the same file name as the provided one.
        /// </summary>
        /// <param name="fileTestBuilder">
        /// Instance of <see cref="IFileTestBuilder"/> type.
        /// </param>
        /// <param name="fileName">File name as string.</param>
        /// <returns>The same <see cref="IAndFileTestBuilder"/>.</returns>
        public static IAndFileTestBuilder WithName(
            this IFileTestBuilder fileTestBuilder,
            string fileName)
        {
            var actualBuilder = GetFileTestBuilder<VirtualFileResult>(fileTestBuilder, FileName);
            
            var virtualFileResult = actualBuilder.ActionResult;
            var actualFileName = virtualFileResult.FileName;

            if (fileName != virtualFileResult.FileName)
            {
                actualBuilder.ThrowNewFailedValidationException(
                    FileName,
                    $"to be '{fileName}'",
                    $"instead received '{actualFileName}'");
            }

            return actualBuilder;
        }

        /// <summary>
        /// Tests whether the <see cref="VirtualFileResult"/>
        /// has the same file provider as the provided one.
        /// </summary>
        /// <param name="fileTestBuilder">
        /// Instance of <see cref="IFileTestBuilder"/> type.
        /// </param>
        /// <param name="fileProvider">File provider of type <see cref="IFileProvider"/>.</param>
        /// <returns>The same <see cref="IAndFileTestBuilder"/>.</returns>
        public static IAndFileTestBuilder WithFileProvider(
            this IFileTestBuilder fileTestBuilder,
            IFileProvider fileProvider)
        {
            var actualBuilder = GetFileTestBuilder<VirtualFileResult>(fileTestBuilder, FileProvider);

            var virtualFileResult = actualBuilder.ActionResult;

            if (fileProvider != virtualFileResult.FileProvider)
            {
                actualBuilder.ThrowNewFailedValidationException(
                    FileProvider,
                    "to be the same as the provided one",
                    "instead received different result");
            }

            return actualBuilder;
        }

        /// <summary>
        /// Tests whether the <see cref="VirtualFileResult"/>
        /// has the same file provider type as the provided one.
        /// </summary>
        /// <param name="fileTestBuilder">
        /// Instance of <see cref="IFileTestBuilder"/> type.
        /// </param>
        /// <returns>The same <see cref="IAndFileTestBuilder"/>.</returns>
        public static IAndFileTestBuilder WithFileProviderOfType<TFileProvider>(
            this IFileTestBuilder fileTestBuilder)
            where TFileProvider : IFileProvider
        {
            var actualBuilder = GetFileTestBuilder<VirtualFileResult>(fileTestBuilder, FileProvider);

            var virtualFileResult = actualBuilder.ActionResult;
            var actualFileProvider = virtualFileResult.FileProvider;

            if (actualFileProvider == null ||
                Reflection.AreDifferentTypes(typeof(TFileProvider), actualFileProvider.GetType()))
            {
                actualBuilder.ThrowNewFailedValidationException(
                    FileProvider,
                    $"to be of {typeof(TFileProvider).Name} type",
                    $"instead received {actualFileProvider.GetName()}");
            }

            return actualBuilder;
        }

        /// <summary>
        /// Tests whether the <see cref="FileContentResult"/>
        /// has the same file contents as the provided byte array.
        /// </summary>
        /// <param name="fileTestBuilder">
        /// Instance of <see cref="IFileTestBuilder"/> type.
        /// </param>
        /// <param name="fileContents">File contents as byte array.</param>
        /// <returns>The same <see cref="IAndFileTestBuilder"/>.</returns>
        public static IAndFileTestBuilder WithContents(
            this IFileTestBuilder fileTestBuilder,
            byte[] fileContents)
        {
            var actualBuilder = GetFileTestBuilder<FileContentResult>(fileTestBuilder, FileContents);

            var fileContentResult = actualBuilder.ActionResult;

            if (!fileContents.SequenceEqual(fileContentResult.FileContents))
            {
                actualBuilder.ThrowNewFailedValidationException(
                    FileContents,
                    "to have values as the provided ones",
                    "instead received different result");
            }

            return actualBuilder;
        }
        
        private static FileTestBuilder<TFileResult> GetFileTestBuilder<TFileResult>(
            IFileTestBuilder fileTestBuilder,
            string containment)
            where TFileResult : FileResult
        {
            var actualFileTestBuilder = fileTestBuilder as FileTestBuilder<TFileResult>;

            if (actualFileTestBuilder == null)
            {
                var fileTestBuilderBase = (BaseTestBuilderWithComponent)fileTestBuilder;

                throw new FileResultAssertionException(string.Format(
                    "{0} file result to contain {1}, but such could not be found.",
                    fileTestBuilderBase.TestContext.ExceptionMessagePrefix,
                    containment));
            }

            return actualFileTestBuilder;
        }
    public static class ContentTestBuilderExtensions
    {
        /// <summary>
        /// Tests whether the <see cref="Microsoft.AspNetCore.Mvc.ContentResult"/>
        /// has the same content as the provided one.
        /// </summary>
        /// <param name="contentTestBuilder">
        /// Instance of <see cref="IContentTestBuilder"/> type.
        /// </param>
        /// <param name="content">Expected content as string.</param>
        /// <returns>The same <see cref="IAndContentTestBuilder"/>.</returns>
        public static IAndContentTestBuilder WithContent(
            this IContentTestBuilder contentTestBuilder,
            string content)
        {
            var actualBuilder = (ContentTestBuilder)contentTestBuilder;

            var actualContent = actualBuilder.ActionResult.Content;

            if (content != actualContent)
            {
                throw ContentResultAssertionException.ForEquality(
                    actualBuilder.TestContext.ExceptionMessagePrefix,
                    content,
                    actualContent);
            }

            return actualBuilder;
        }

        /// <summary>
        /// Tests whether the <see cref="Microsoft.AspNetCore.Mvc.ContentResult"/>
        /// has content passing the given assertions.
        /// </summary>
        /// <param name="contentTestBuilder">
        /// Instance of <see cref="IContentTestBuilder"/> type.
        /// </param>
        /// <param name="assertions">Action containing all assertions for the content.</param>
        /// <returns>The same <see cref="IAndContentTestBuilder"/>.</returns>
        public static IAndContentTestBuilder WithContent(
            this IContentTestBuilder contentTestBuilder,
            Action<string> assertions)
        {
            var actualBuilder = (ContentTestBuilder)contentTestBuilder;

            var actualContent = actualBuilder.ActionResult.Content;

            assertions(actualContent);

            return actualBuilder;
        }

        /// <summary>
        /// Tests whether the <see cref="Microsoft.AspNetCore.Mvc.ContentResult"/>
        /// has content passing the given predicate.
        /// </summary>
        /// <param name="contentTestBuilder">
        /// Instance of <see cref="IContentTestBuilder"/> type.
        /// </param>
        /// <param name="predicate">Predicate testing the content.</param>
        /// <returns>The same <see cref="IAndContentTestBuilder"/>.</returns>
        public static IAndContentTestBuilder WithContent(
            this IContentTestBuilder contentTestBuilder,
            Func<string, bool> predicate)
        {
            var actualBuilder = (ContentTestBuilder)contentTestBuilder;

            var actualContent = actualBuilder.ActionResult.Content;

            if (!predicate(actualContent))
            {
                throw ContentResultAssertionException.ForPredicate(
                    actualBuilder.TestContext.ExceptionMessagePrefix,
                    actualContent);
            }

            return actualBuilder;
        }
     public static class ContentTestBuilderExtensions
    {
        /// <summary>
        /// Tests whether the <see cref="Microsoft.AspNetCore.Mvc.ContentResult"/>
        /// has the same content as the provided one.
        /// </summary>
        /// <param name="contentTestBuilder">
        /// Instance of <see cref="IContentTestBuilder"/> type.
        /// </param>
        /// <param name="content">Expected content as string.</param>
        /// <returns>The same <see cref="IAndContentTestBuilder"/>.</returns>
        public static IAndContentTestBuilder WithContent(
            this IContentTestBuilder contentTestBuilder,
            string content)
        {
            var actualBuilder = (ContentTestBuilder)contentTestBuilder;

            var actualContent = actualBuilder.ActionResult.Content;

            if (content != actualContent)
            {
                throw ContentResultAssertionException.ForEquality(
                    actualBuilder.TestContext.ExceptionMessagePrefix,
                    content,
                    actualContent);
            }

            return actualBuilder;
        }

        /// <summary>
        /// Tests whether the <see cref="Microsoft.AspNetCore.Mvc.ContentResult"/>
        /// has content passing the given assertions.
        /// </summary>
        /// <param name="contentTestBuilder">
        /// Instance of <see cref="IContentTestBuilder"/> type.
        /// </param>
        /// <param name="assertions">Action containing all assertions for the content.</param>
        /// <returns>The same <see cref="IAndContentTestBuilder"/>.</returns>
        public static IAndContentTestBuilder WithContent(
            this IContentTestBuilder contentTestBuilder,
            Action<string> assertions)
        {
            var actualBuilder = (ContentTestBuilder)contentTestBuilder;

            var actualContent = actualBuilder.ActionResult.Content;

            assertions(actualContent);

            return actualBuilder;
        }

        /// <summary>
        /// Tests whether the <see cref="Microsoft.AspNetCore.Mvc.ContentResult"/>
        /// has content passing the given predicate.
        /// </summary>
        /// <param name="contentTestBuilder">
        /// Instance of <see cref="IContentTestBuilder"/> type.
        /// </param>
        /// <param name="predicate">Predicate testing the content.</param>
        /// <returns>The same <see cref="IAndContentTestBuilder"/>.</returns>
        public static IAndContentTestBuilder WithContent(
            this IContentTestBuilder contentTestBuilder,
            Func<string, bool> predicate)
        {
            var actualBuilder = (ContentTestBuilder)contentTestBuilder;

            var actualContent = actualBuilder.ActionResult.Content;

            if (!predicate(actualContent))
            {
                throw ContentResultAssertionException.ForPredicate(
                    actualBuilder.TestContext.ExceptionMessagePrefix,
                    actualContent);
            }

            return actualBuilder;
        }
    }
}

